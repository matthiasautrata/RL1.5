# Adalbert Issues Tracker

Systematic review findings from cross-LLM audit. Issues ordered by severity.

---

## Preliminary: DCON Alignment File

The former `ontology/adalbert-dcon-alignment.ttl` mapped Adalbert back to DCON via `skos:closeMatch`.

**Decision (v0.7):** Adalbert supersedes DCON. The alignment file has been deleted — `docs/comparisons/comparison-dcon.md` serves as the complete supersession record. DCON's promise hierarchy dissolves into `odrl:Duty` patterns with `adalbert:recurrence` for scheduling. Time handling is covered by `recurrence` (RRULE scheduling) + `deadline` (fulfillment window). The `../dcon` folder remains separate — it is not migrated.

**Status:** Resolved (v0.7)

---

## Critical

### Issue 11: Deadline type mismatch between semantics and ontology/SHACL/examples

The formal semantics (`docs/Adalbert_Semantics.md`) defines `Deadline` as either an absolute `dateTime` or a relative `duration`. The ontology, SHACL, and documentation explicitly allowed `xsd:time` (daily recurring deadline), and examples used it (e.g., `"07:30:00"^^xsd:time`). This created a direct conflict between the normative semantics and the rest of the stack.

| Artifact | Location |
|----------|----------|
| Formal semantics | `docs/Adalbert_Semantics.md` |
| Ontology | `ontology/adalbert-core.ttl` |
| SHACL | `ontology/adalbert-shacl.ttl` |
| Documentation | `docs/adalbert-specification.md` |
| Examples | `examples/baseline.ttl` |

**Decision:** Alternative B — Remove `xsd:time` from ontology/SHACL/docs/examples. Model daily windows using `recurrence` plus `xsd:duration` deadlines only. For example, `"FREQ=DAILY;BYHOUR=7;BYMINUTE=0"` with `"PT30M"^^xsd:duration` means "generate duty at 07:00, fulfill within 30 minutes" (effective deadline 07:30). This preserves all functional capability while aligning with the formal semantics.

**Changes:**

- `ontology/adalbert-core.ttl`: Removed xsd:time from deadline property comment
- `ontology/adalbert-shacl.ttl`: Removed xsd:time from deadline datatype SHACL constraint
- `docs/adalbert-specification.md`: Removed xsd:time from deadline types table and SHACL summary
- `examples/baseline.ttl`: Changed `"07:30:00"^^xsd:time` to `"PT30M"^^xsd:duration`
- `examples/data-contract.ttl`: Changed `"06:30:00"^^xsd:time` to `"PT30M"^^xsd:duration`
- `examples/README.md`: Updated example to use duration
- `docs/README.md`: Removed xsd:time from deadline semantics section
- `docs/adalbert-overview.md`: Updated example to use duration
- `docs/adalbert-term-mapping.md`: Removed xsd:time from datatype list and examples
- `docs/contracts-guide.md`: Updated all examples and checklist to remove xsd:time
- `docs/comparisons/comparison-dcon.md`: Updated Adalbert example to use duration

**Status:** Resolved (v0.7.1)

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

**Decision:** Option A — declare `rdfs:subPropertyOf odrl:partOf` on both properties. This is the standard ODRL profile pattern: 2 lines of RDF, zero impact on semantics or examples. ODRL processors with RDFS reasoning can infer `odrl:partOf` from Adalbert triples. Transitivity and typed domains remain Adalbert-specific additions not inherited by the base property.

**Changes:**

- `ontology/adalbert-core.ttl`: Added `rdfs:subPropertyOf odrl:partOf` to both `adalbert:partOf` and `adalbert:memberOf`; updated `rdfs:comment` to note the bridge; updated `dcterms:modified`
- `docs/adalbert-specification.md`: Added `SubPropertyOf` row to property tables (§4.7, §4.8); updated AssetCollection/PartyCollection reasons in §8
- `docs/comparisons/comparison-odrl22.md`: Added ODRL Bridge column to hierarchy table; added bridge explanation paragraph; updated AssetCollection/PartyCollection exclusion reasons
- `docs/conformance-w3c-best-practices.md`: Updated party functions and asset relationships sections
- `docs/README.md`: Added parenthetical `rdfs:subPropertyOf odrl:partOf` to extension list

**Status:** Resolved (v0.7.1)

---

## Medium

### Issue 12: Docs require `odrl:conflict`, SHACL does not enforce it

The policy checklist requires `odrl:conflict odrl:prohibit`, but `adalbertsh:PolicyShape` does not enforce a conflict value. This allows policies to pass SHACL while violating the documented requirement.

| Artifact | Location |
|----------|----------|
| SHACL | `ontology/adalbert-shacl.ttl` |
| Docs | `docs/policy-writers-guide.md` |

**Decision:** Option C — declare once at profile level, relax per-policy requirement. Since Adalbert hardcodes Prohibition > Permission with no configurable alternative, per-policy declaration is ceremony. The conflict strategy is now declared once in `adalbert-prof.ttl` and inherited by all policies.

**Changes:**

- `ontology/adalbert-prof.ttl`: Added `odrl:conflict odrl:prohibit` to the Adalbert Core profile declaration
- `ontology/adalbert-shacl.ttl`: Removed `odrl:conflict` property constraint from `PolicyShape`
- `ontology/adalbert-core.ttl`: Updated comment to reference profile-level declaration
- `docs/adalbert-specification.md`: Updated PolicyShape description and profile constraints table
- `docs/policy-writers-guide.md`: Updated checklist item 2; removed `odrl:conflict odrl:prohibit` from 2 example policies
- `docs/contracts-guide.md`: Updated checklist item 2; removed `odrl:conflict odrl:prohibit` from 2 example contracts
- `examples/baseline.ttl`: Removed 11 occurrences of `odrl:conflict odrl:prohibit`
- `examples/data-contract.ttl`: Removed 2 occurrences
- `examples/data-use-policy.ttl`: Removed 2 occurrences

**Status:** Resolved (v0.7.1)

### Issue 13: DUE profile version out of sync

`profiles/adalbert-due.ttl` declares version `0.6` while core ontology and documentation are at `0.7`. This causes release confusion for downstream tooling.

| Artifact | Location |
|----------|----------|
| DUE profile | `profiles/adalbert-due.ttl` |

**Decision:** Update DUE profile version info to `0.7` for consistency.

**Changes:**

- `profiles/adalbert-due.ttl`: Updated header comment and `owl:versionInfo` from `0.6` to `0.7`

**Status:** Resolved (v0.7.1)

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

`docs/README.md` omitted `adalbert:memberOf` from the extension list. (`rightOperandRef` was removed in Issue 3, so its absence is now correct.) The file also previously excluded `xsd:time` from deadline semantics, which was correct per Issue 11 resolution.

**Fix:** Added `memberOf` to extension list. `xsd:time` was later removed from the entire stack per Issue 11 (Alternative B).

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

### Issue 14: Redundant `odrl:target` in examples and guides

Examples repeated `odrl:target` at both policy and rule levels for the same asset. This was valid but noisy and made examples harder to read.

| Artifact | Location |
|----------|----------|
| Examples | `examples/baseline.ttl`, `examples/data-contract.ttl`, `examples/data-use-policy.ttl` |
| Docs | `docs/contracts-guide.md`, `docs/policy-writers-guide.md` |

**Decision:** Option A — adopt ODRL's standard target inheritance pattern. Policy-level `odrl:target` is inherited by rules unless overridden at rule level. Rules targeting a different asset declare their own target. Multi-target policies require explicit rule-level targets (inheritance is ambiguous).

**Changes:**

- `examples/data-contract.ttl`: Removed redundant rule-level targets matching policy target; kept targets on duties with different assets (schema-changes, usage-stats)
- `examples/data-use-policy.ttl`: Same pattern — removed redundant targets on permissions/prohibitions/duties matching policy target; kept access-log target on logging duty
- `examples/baseline.ttl`: Removed redundant targets from 8 single-target contracts and 2 subscriptions; kept all targets in multi-asset contract (Contract 8) and on duties with different assets
- `docs/contracts-guide.md`: Updated examples with target inheritance; added Target Inheritance subsection in Advanced Topics; documented multi-target rule
- `docs/policy-writers-guide.md`: Updated checklist item 4 to note target inheritance

**Status:** Resolved (v0.7.2)

### Issue 15: Resolution paths not enforced for DUE operands

SHACL allows `adalbert:resolutionPath` to be absent on any `odrl:LeftOperand`. For DUE operands, resolution paths are required by convention but not validated, so malformed operands can pass validation.

| Artifact | Location |
|----------|----------|
| SHACL | `ontology/adalbert-shacl.ttl` |
| DUE profile | `profiles/adalbert-due.ttl` |

**Decision needed:** Add a DUE-specific shape that requires `adalbert:resolutionPath` for all operands in the DUE namespace.

**Status:** Open

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
| Q3 | Keep custom `partOf`/`memberOf`, or converge on `odrl:partOf` with additional constraints? | Issue 4 | Resolved: Option A — keep custom properties, bridge via `rdfs:subPropertyOf odrl:partOf` |
| Q4 | Is `odrl:assignee` appropriate on duties? Should there be dedicated duty party roles? | Resolved: `adalbert:subject` (`rdfs:subPropertyOf odrl:assignee`) replaces `odrl:assignee` on duties — avoids role overloading where provider is both assigner and assignee |
| Q5 | How to express "to whom" on duties like notify/report? Is `odrl:target` appropriate for the affected party? | Resolved: `adalbert:object` (`rdfs:subPropertyOf odrl:function`) for affected party; `odrl:target` stays for assets. Aligns with MDS md:subject/md:object and rl2:subject/counterparty |


---

## Resolution Log

| Date | Issue | Action | Commit |
|------|-------|--------|--------|
| 2026-02-04 | Issue 1, Q1 | Separated concerns: `currentAgent` for constraints only (RL2-aligned); absent assignee = universal applicability; removed from 11 example rules | -- |
| 2026-02-04 | Issue 2 | Added RuntimeRef fallback to `resolve()` — dual-typed operands delegate to `resolveRuntime()` | -- |
| 2026-02-04 | Issue 3, Q2 | Removed `rightOperandRef` — dead code, identity binding deferred to RL2; simplified SHACL and abstract syntax | -- |
| 2026-02-04 | Issue 5 | Replaced `adalbert-due:purpose` with `odrl:purpose` + resolutionPath — standard ODRL profile pattern; updated 7 files | -- |
| 2026-02-04 | Issue 6 | Added `odrl:includedIn` hierarchy to 7 DUE actions; documented `conformTo` exception | -- |
| 2026-02-04 | Issue 7 | Fixed three doc inconsistencies: added `memberOf` to README, fixed `dcon:Duty` → `dcon:Promise`, removed false SKOS claims | -- |
| 2026-02-06 | Issue 7 (revert) | Removed `xsd:time` from README per Issue 11 resolution (Alternative B) | -- |
| 2026-02-04 | Issue 8 | Added `adalbertsh:SetShape` requiring `odrl:target` on Set policies | -- |
| 2026-02-04 | Issue 9 | Added DUE profile declaration to all 4 example policy instances | -- |
| 2026-02-04 | Issue 10 | Restructured RL2 comparison with full-stack footprint (ODRL base + extensions vs RL2 standalone) | -- |
| 2026-02-06 | Issue 11 | Removed `xsd:time` from deadline — Alternative B: use recurrence + duration for daily windows; updated 11 files (ontology, SHACL, 7 docs, 3 examples) | -- |
| 2026-02-06 | Issue 12 | Moved `odrl:conflict` to profile level (Option C) — declared once in `adalbert-prof.ttl`; removed from SHACL, examples, guides; 9 files modified | -- |
| 2026-02-06 | Issue 13 | Updated DUE profile version from `0.6` to `0.7` — synced with core ontology version | -- |
| 2026-02-06 | Issue 4, Q3 | Option A: declared `rdfs:subPropertyOf odrl:partOf` on both `adalbert:partOf` and `adalbert:memberOf` — standard ODRL profile bridge pattern; updated ontology, spec, comparison, conformance, README | -- |
| 2026-02-06 | Q4, Q5 | Added `adalbert:subject` (rdfs:subPropertyOf odrl:assignee) and `adalbert:object` (rdfs:subPropertyOf odrl:function) for duty party roles; replaced `odrl:assignee` on all duty instances; updated ontology, SHACL, all examples, 7 docs | -- |
| 2026-02-06 | Issue 14 | Adopted ODRL target inheritance — policy-level target inherited by rules unless overridden; removed redundant targets from examples and guides | -- |
| 2026-02-04 | DCON Supersession | v0.7: DCON superseded; added `adalbert:recurrence`; deprecated alignment file; rewrote comparison-dcon.md; created contracts-guide.md; 16 files modified, 1 created | -- |
