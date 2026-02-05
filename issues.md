# Adalbert Issues Tracker

Systematic review findings from cross-LLM audit. Issues ordered by severity.

---

## Preliminary: DCON Alignment File

The former `ontology/adalbert-dcon-alignment.ttl` mapped Adalbert back to DCON via `skos:closeMatch`.

**Decision (v0.7):** Adalbert supersedes DCON. The alignment file has been deleted — `docs/comparisons/comparison-dcon.md` serves as the complete supersession record. DCON's promise hierarchy dissolves into `odrl:Duty` patterns with `adalbert:recurrence` for scheduling. Time handling is covered by `recurrence` (RRULE scheduling) + `deadline` (fulfillment window). The `../dcon` folder remains separate — it is not migrated.

**Status:** Resolved (v0.7)

---

## Critical

### Issue 1: Runtime reference `currentAgent` inconsistent across artifacts

`adalbert:currentAgent` was used as `odrl:assignee` in examples, but the formal semantics only matched `Env.agent`, wildcard `*`, or `memberOf+` and never treated `currentAgent` as a special case. This made many example rules non-applicable under the semantics.

**Decision:** Neither wildcard alias nor runtime substitution. Separated the concerns:

- `currentAgent` is for constraint comparisons (identity binding) only, matching RL2's design
- Rules that apply to any requesting agent omit `odrl:assignee` (standard ODRL for Set/Offer)
- Agreements/Subscriptions use concrete party URIs (already the case)
- NormActive now treats absent assignee (`norm.subject = ⊥`) as universal applicability

This aligns with ODRL's Set policy rules (Sets should not bind parties) and with RL2's usage of `currentAgent` exclusively in `rightOperandRef` for identity binding.

**Changes:**

- `docs/Adalbert_Semantics.md`: Added `norm.subject = ⊥` disjunct to NormActive
- `ontology/adalbert-core.ttl`: Updated `currentAgent` comment (constraint use only)
- `examples/data-use-policy.ttl`: Removed `odrl:assignee adalbert:currentAgent` from 6 rules
- `examples/data-contract.ttl`: Removed `odrl:assignee adalbert:currentAgent` from 5 Offer rules
- `docs/comparisons/comparison-dcon.md`: Removed `currentAgent` from example Offer

**Status:** Resolved

---

### Issue 2: `currentDateTime` as left operand always evaluates false

`adalbert:currentDateTime` was used as a left operand in examples, but `resolve()` only handled path-based resolution and fell back to ⊥. Since `currentDateTime` has no `resolutionPath`, those constraints always evaluated false. The SLA duty never activated.

**Decision:** Add RuntimeRef fallback to `resolve()`. The ontology already dual-types `currentDateTime` as both `odrl:LeftOperand` and `adalbert:RuntimeReference`. The `resolveRuntime()` function already maps `currentDateTime → Env.Σ.clock`. The `resolve()` function now checks for RuntimeReference typing between path-based resolution and the ⊥ fallback, delegating to `resolveRuntime()`. Path-based resolution retains precedence.

The correct long-term solution would be a `state.*` resolution root (as RL2 provides), but that is RL2 territory and would complicate Adalbert's simpler model. The RuntimeRef fallback is the minimal compromise.

**Changes:**

- `docs/Adalbert_Semantics.md`: Added `op ∈ RuntimeRef → resolveRuntime(op, Env)` case to `resolve()`; updated architectural principle text

**Status:** Resolved

---

## High

### Issue 3: `rightOperandRef` underspecified and duplicates ODRL

`adalbert:rightOperandRef` was defined in the ontology and enforced via SHACL xone, but had zero uses in examples or documentation. The semantics allowed `RuntimeRef` in `rightOperand` but no RDF mapping existed. ODRL's `odrl:rightOperandReference` serves a different purpose (web-resource dereferencing, not evaluation-engine resolution).

**Decision:** Remove `rightOperandRef` entirely. The property was dead code -- Adalbert has no identity binding patterns (tun-sollen, separation of duty), which are the only use case for runtime references in right-operand position. Identity binding is explicitly deferred to RL2 (`rl2:rightOperandRef`). ODRL's `rightOperandReference` is not a replacement -- it has different semantics (HTTP dereferencing vs engine resolution).

**Changes:**

- `ontology/adalbert-core.ttl`: Removed `adalbert:rightOperandRef` property
- `ontology/adalbert-shacl.ttl`: Replaced xone with simple `odrl:rightOperand` minCount 1
- `docs/Adalbert_Semantics.md`: Simplified abstract syntax (`rightOperand: Value`), removed `RuntimeRef` case from constraint evaluation, updated `resolveRuntime` description
- `LLM.md`: Removed from extension table
- `config/namespaces.ttl`: Removed from namespace comment
- `docs/comparisons/namespace-alignment.md`: Removed from extension lists
- `docs/comparisons/comparison-odrl22.md`: Removed from runtime reference table
- `docs/comparisons/comparison-rl2.md`: Updated mapping table (identity binding added at RL2)

**Status:** Resolved

---

### Issue 4: `partOf` and `memberOf` duplicate ODRL's `odrl:partOf`

Adalbert defines two custom hierarchy properties where ODRL already provides one:

- `adalbert:partOf` -- domain: `odrl:Asset`, range: `odrl:Asset`, transitive
- `adalbert:memberOf` -- domain: `odrl:Party`, range: `odrl:Party`, transitive
- `odrl:partOf` -- domain: Asset∪Party, range: AssetCollection∪PartyCollection, **not transitive**

| Artifact | Location |
|----------|----------|
| Core ontology | `ontology/adalbert-core.ttl:178-190` |
| Formal semantics | `docs/Adalbert_Semantics.md:382-393, 669, 876-892` |
| Examples | `examples/data-use-policy.ttl:20,34,38`, `examples/data-contract.ttl:24,35` |

**The differences are real, not cosmetic:**

| Aspect | ODRL `partOf` | Adalbert `partOf`/`memberOf` |
|--------|---------------|------------------------------|
| Transitivity | Not declared | Explicit `owl:TransitiveProperty` |
| Domain separation | One property, both domains | Separate properties, typed domains |
| Range | AssetCollection/PartyCollection | Plain Asset/Party |
| Collection model | Requires Collection subclasses + refinement | Direct hierarchy, no collection classes |

Adalbert's semantics relies on transitive closure: `partOf⁺` in asset matching, `memberOf⁺` in NormActive and agent matching, and `performed()` checks. This is core to the evaluation model.

**RL2 comparison:** RL2 uses neither `odrl:partOf` nor Adalbert's properties. RL2 has `rl2:member` (reversed direction, collection→asset) and `roles()` for party matching. No alignment concern.

**Alternatives (in preference order):**

**A. Declare `rdfs:subPropertyOf odrl:partOf` (recommended).** Add two lines to the ontology. Preserves transitivity, domain typing, and evaluation semantics. ODRL processors with RDFS reasoning can infer `odrl:partOf` from Adalbert triples. This is what ODRL profiles are designed to do: specialize base vocabulary terms.

**B. Replace with `odrl:partOf` directly.** High disruption. ODRL's `partOf` is not transitive -- asserting transitivity on a base vocabulary term may break ODRL processors. Loses domain separation (no type-checking that parties and assets aren't mixed). Changes ontology, all examples, semantics, SHACL.

**C. Replace `adalbert:partOf` only (assets), keep `adalbert:memberOf` (parties).** Partial. Same transitivity problem for assets. Inconsistent principle.

**D. Keep as-is.** The properties aren't redundant (they add transitivity and domain typing). But without subproperty bridging, ODRL processors can't interpret Adalbert hierarchies at all.

**Decision needed:** Which alternative? Requires discussion -- affects ODRL interoperability posture.

**Status:** Open

---

## Medium

### Issue 5: DUE `purpose` duplicates ODRL purpose operand

DUE defines `adalbert-due:purpose`, but ODRL already provides a `purpose` left operand. ODRL also defines a `recipient` left operand that may cover `recipientType`. This creates avoidable duplication and reduces portability.

| Artifact | Location |
|----------|----------|
| DUE profile | `profiles/adalbert-due.ttl:63-67, 224-230` |

**Decision:** Replace `adalbert-due:purpose` with `odrl:purpose` extended with `adalbert:resolutionPath "context.purpose"`. This is the standard ODRL profile pattern — profiles annotate existing ODRL operands rather than redefining them. `recipientType` is retained as genuinely different from `odrl:recipient` (classification vs party identity).

**Changes:**

- `profiles/adalbert-due.ttl`: Replaced `adalbert-due:purpose` definition with `odrl:purpose adalbert:resolutionPath "context.purpose"`
- `examples/data-use-policy.ttl`: Updated 4 references from `adalbert-due:purpose` to `odrl:purpose`
- `examples/README.md`: Updated 1 reference
- `docs/Adalbert_Semantics.md`: Updated example in §11.1 to show both extending existing ODRL operands and declaring new ones
- `docs/conformance-w3c-best-practices.md`: Updated operand reference
- `docs/comparisons/comparison-odrl-profiles.md`: Updated example
- `docs/comparisons/comparison-w3c-market-data.md`: Updated mapping table

**Status:** Resolved

---

### Issue 6: DUE actions missing `odrl:includedIn` hierarchy

Several DUE actions lack `odrl:includedIn` declarations: `conformTo`, `log`, `notify`, `report`, `deliver`, `export`, `copy`, `link`. The ODRL profile mechanism expects new actions to declare a parent via `odrl:includedIn`. Missing hierarchy weakens reasoning and profile conformance.

| Artifact | Location |
|----------|----------|
| DUE profile | `profiles/adalbert-due.ttl:351-407` |

**Decision:** Add `odrl:includedIn` where semantically sound (7 of 8 actions). One exception documented.

| Action | Parent | Rationale |
|--------|--------|-----------|
| `log` | `odrl:inform` | Logging is a specific form of informing |
| `notify` | `odrl:inform` | Direct match |
| `report` | `odrl:inform` | Reporting is a form of informing |
| `deliver` | `odrl:distribute` | Delivering is supplying data |
| `export` | `odrl:distribute` | Exporting is distributing externally |
| `copy` | `odrl:reproduce` | Making duplicates |
| `link` | `odrl:aggregate` | Linking/joining is aggregating |
| `conformTo` | *(none)* | Governance action with no natural ODRL parent |

**Changes:**

- `profiles/adalbert-due.ttl`: Added `odrl:includedIn` to 7 actions; documented `conformTo` exception

**Status:** Resolved

---

### Issue 7: Documentation inconsistencies (three sub-issues)

#### 7a: `docs/README.md` omits extensions and deadline type

`docs/README.md` omits `adalbert:memberOf` from the extension list and excludes `xsd:time` from deadline semantics, but core ontology and SHACL allow it. (`rightOperandRef` was removed in Issue 3, so its absence is now correct.)

**Fix:** Added `memberOf` to extension list; added `xsd:time` to deadline types.

#### 7b: Duty/Promise mapping inconsistency

`profiles/README.md` maps `odrl:Duty` to `dcon:Duty`, while the alignment file and comparison doc use `dcon:Promise`. The alignment file is authoritative.

**Fix:** Changed `profiles/README.md` from `dcon:Duty` to `dcon:Promise`.

#### 7c: Conformance doc claims unsupported features

`docs/conformance-w3c-best-practices.md` claims `skos:ConceptScheme` typing and `skos:Collection` structures that do not exist in the ontology.

**Fix:** Removed `skos:ConceptScheme` from profile type assertion; replaced SKOS Collection example with a note that SKOS structuring is planned but not yet implemented.

**Changes:**

- `docs/README.md`: Added `memberOf`, added `xsd:time` deadline type
- `profiles/README.md`: Fixed `dcon:Duty` → `dcon:Promise`
- `docs/conformance-w3c-best-practices.md`: Removed false `skos:ConceptScheme` claim; replaced aspirational SKOS Collection example with honest status note

**Status:** Resolved

---

## Low

### Issue 8: SHACL missing `odrl:Set` target constraint

SHACL does not enforce ODRL's requirement that an `odrl:Set` contains an `odrl:target`. Adding a Set shape would tighten profile validation.

**Decision:** Added `adalbertsh:SetShape` requiring at least one `odrl:target` on Set policies.

**Changes:**

- `ontology/adalbert-shacl.ttl`: Added SetShape with `odrl:target` minCount 1

**Status:** Resolved

---

### Issue 9: Example policies missing DUE profile declaration

Example policies use DUE vocabulary but only declare the core profile in `odrl:profile`. ODRL allows multiple profile identifiers; both core and DUE should be listed.

| Artifact | Location |
|----------|----------|
| Data use policy | `examples/data-use-policy.ttl:34-37` |
| Data contract | `examples/data-contract.ttl:55-61` |

**Decision:** Added DUE profile declaration alongside core profile in all four policy instances (2 in each example file).

**Changes:**

- `examples/data-use-policy.ttl`: Added `<https://vocabulary.bigbank/adalbert/due/>` to 2 policies
- `examples/data-contract.ttl`: Added `<https://vocabulary.bigbank/adalbert/due/>` to 2 policies

**Status:** Resolved

---

### Issue 10: Update the docs/comparisons/comparison-rl2.md

Consider that RL2 is self-sufficient and RL1.5 lives ontop of ODRL2.2. So, a fair comparison would include the full foot-print in both cases.

**Decision:** Restructured executive summary and §8 (Complexity Assessment) with full-stack footprint analysis. Adalbert uses 22 classes + 38 properties (13 native + 47 ODRL); RL2 uses ~43 classes + ~55 properties (standalone). The size gap narrows from naive 4.8x to 1.5-2x when counting the full stack. Implementation complexity remains 3-4x due to fundamentally new evaluation mechanics in RL2.

**Changes:**

- `docs/comparisons/comparison-rl2.md`: Updated executive summary table with effective vocabulary counts; restructured §8 with layered footprint breakdown (ODRL base, Adalbert extensions, DUE profile vs RL2 core, RL2 protocol) and side-by-side comparison

**Status:** Resolved

## Open Questions / Decisions

These require human input before resolution:

| # | Question | Related Issues | Status |
|---|----------|----------------|--------|
| Q1 | Should `adalbert:currentAgent` be the wildcard agent (`*`) or a runtime substitution for `odrl:assignee`? | Issue 1 | Resolved: neither -- omit assignee for universal rules, keep `currentAgent` for constraints only |
| Q2 | Keep `adalbert:rightOperandRef`, or switch to `odrl:rightOperandReference` with explicit RDF mapping? | Issue 3 | Resolved: removed -- dead code, identity binding deferred to RL2 |
| Q3 | Keep custom `partOf`/`memberOf`, or converge on `odrl:partOf` with additional constraints? | Issue 4 | Open |

---

## Resolution Log

| Date | Issue | Action | Commit |
|------|-------|--------|--------|
| 2026-02-04 | Issue 1, Q1 | Separated concerns: `currentAgent` for constraints only (RL2-aligned); absent assignee = universal applicability; removed from 11 example rules | -- |
| 2026-02-04 | Issue 2 | Added RuntimeRef fallback to `resolve()` — dual-typed operands delegate to `resolveRuntime()` | -- |
| 2026-02-04 | Issue 3, Q2 | Removed `rightOperandRef` — dead code, identity binding deferred to RL2; simplified SHACL and abstract syntax | -- |
| 2026-02-04 | Issue 5 | Replaced `adalbert-due:purpose` with `odrl:purpose` + resolutionPath — standard ODRL profile pattern; updated 7 files | -- |
| 2026-02-04 | Issue 6 | Added `odrl:includedIn` hierarchy to 7 DUE actions; documented `conformTo` exception | -- |
| 2026-02-04 | Issue 7 | Fixed three doc inconsistencies: added `memberOf` + `xsd:time` to README, fixed `dcon:Duty` → `dcon:Promise`, removed false SKOS claims | -- |
| 2026-02-04 | Issue 8 | Added `adalbertsh:SetShape` requiring `odrl:target` on Set policies | -- |
| 2026-02-04 | Issue 9 | Added DUE profile declaration to all 4 example policy instances | -- |
| 2026-02-04 | Issue 10 | Restructured RL2 comparison with full-stack footprint (ODRL base + extensions vs RL2 standalone) | -- |
| 2026-02-04 | DCON Supersession | v0.7: DCON superseded; added `adalbert:recurrence`; deprecated alignment file; rewrote comparison-dcon.md; created contracts-guide.md; 16 files modified, 1 created | -- |
