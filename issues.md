# Adalbert Issues Tracker

Systematic review findings from cross-LLM audit. Issues ordered by severity.

---

## Preliminary: DCON Alignment File

`ontology/adalbert-dcon-alignment.ttl` maps Adalbert back to DCON via `skos:closeMatch`. If DCON absorption is complete and runtime interoperability with legacy DCON systems is not required, this file is documentation/metadata only. It can remain for reference or move to `docs/`. It is not strictly redundant -- it provides semantic bridging.

**Decision needed:** Keep in `ontology/`, move to `docs/`, or remove?
**Status:** Open

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

`adalbert:currentDateTime` is used as a left operand in examples, but left operands only resolve via `adalbert:resolutionPath` and otherwise return bottom. Since `currentDateTime` lacks a resolution path, those constraints always evaluate false. The SLA duty in the data contract example never activates.

| Artifact | Location |
|----------|----------|
| Formal semantics | `docs/Adalbert_Semantics.md:441-461` |
| Core ontology | `ontology/adalbert-core.ttl:226-229` |
| Data contract example | `examples/data-contract.ttl:68-80` |

**Status:** Open

---

## High

### Issue 3: `rightOperandRef` underspecified and duplicates ODRL

The semantics allows `RuntimeRef` in `rightOperand`, but there is no RDF mapping to `adalbert:rightOperandRef`. SHACL enforces `rightOperand` xor `adalbert:rightOperandRef`. Meanwhile, ODRL already defines `odrl:rightOperandReference` as the standard alternative to `odrl:rightOperand`, so a custom property weakens interoperability.

| Artifact | Location |
|----------|----------|
| Formal semantics | `docs/Adalbert_Semantics.md:157-174, 552-563` |
| Core ontology | `ontology/adalbert-core.ttl:231-235` |
| SHACL shapes | `ontology/adalbert-shacl.ttl:183-188` |

**Decision needed:** Keep `adalbert:rightOperandRef`, or switch to `odrl:rightOperandReference` with explicit RDF mapping?

**Status:** Open

---

### Issue 4: `partOf` and `memberOf` duplicate ODRL's `odrl:partOf`

`adalbert:partOf` and `adalbert:memberOf` duplicate ODRL's existing `odrl:partOf` for asset/party membership. This directly contradicts the "use ODRL terms directly" principle.

| Artifact | Location |
|----------|----------|
| Core ontology | `ontology/adalbert-core.ttl:179-191` |

**Decision needed:** Converge on `odrl:partOf` with additional SHACL constraints, or keep custom properties with `rdfs:subPropertyOf odrl:partOf`?

**Status:** Open

---

## Medium

### Issue 5: DUE `purpose` duplicates ODRL purpose operand

DUE defines `adalbert-due:purpose`, but ODRL already provides a `purpose` left operand. ODRL also defines a `recipient` left operand that may cover `recipientType`. This creates avoidable duplication and reduces portability.

| Artifact | Location |
|----------|----------|
| DUE profile | `profiles/adalbert-due.ttl:63-67, 224-230` |

**Status:** Open

---

### Issue 6: DUE actions missing `odrl:includedIn` hierarchy

Several DUE actions lack `odrl:includedIn` declarations: `conformTo`, `log`, `notify`, `report`, `deliver`, `export`, `copy`, `link`. The ODRL profile mechanism expects new actions to declare a parent via `odrl:includedIn`. Missing hierarchy weakens reasoning and profile conformance.

| Artifact | Location |
|----------|----------|
| DUE profile | `profiles/adalbert-due.ttl:351-407` |

**Status:** Open

---

### Issue 7: Documentation inconsistencies (three sub-issues)

#### 7a: `docs/README.md` omits extensions and deadline type

`docs/README.md` omits `adalbert:memberOf` and `adalbert:rightOperandRef` from the extension list and excludes `xsd:time` from deadline semantics, but core ontology and SHACL allow it.

| Artifact | Location |
|----------|----------|
| README | `docs/README.md:32-52` |
| Core ontology | `ontology/adalbert-core.ttl:111-124` |
| SHACL shapes | `ontology/adalbert-shacl.ttl:148-156` |

#### 7b: Duty/Promise mapping inconsistency

`profiles/README.md` maps `odrl:Duty` to `dcon:Duty`, while the alignment and comparison docs use `dcon:Promise`. These should be consistent.

| Artifact | Location |
|----------|----------|
| Profiles README | `profiles/README.md:73-78` |
| DCON alignment | `ontology/adalbert-dcon-alignment.ttl:57-63` |
| DCON comparison | `docs/comparisons/comparison-dcon.md:9-13` |

#### 7c: Conformance doc claims unsupported features

`docs/conformance-w3c-best-practices.md` claims SKOS ConceptScheme/Collections and `odrl:purpose` usage that are not present in the ontology.

| Artifact | Location |
|----------|----------|
| Conformance doc | `docs/conformance-w3c-best-practices.md:19-55` |

**Status:** Open

---

## Low

### Issue 8: SHACL missing `odrl:Set` target constraint

SHACL does not enforce ODRL's requirement that an `odrl:Set` contains an `odrl:target`. Adding a Set shape would tighten profile validation.

**Status:** Open

---

### Issue 9: Example policies missing DUE profile declaration

Example policies use DUE vocabulary but only declare the core profile in `odrl:profile`. ODRL allows multiple profile identifiers; both core and DUE should be listed.

| Artifact | Location |
|----------|----------|
| Data use policy | `examples/data-use-policy.ttl:34-37` |
| Data contract | `examples/data-contract.ttl:55-61` |

**Status:** Open

---

## Open Questions / Decisions

These require human input before resolution:

| # | Question | Related Issues | Status |
|---|----------|----------------|--------|
| Q1 | Should `adalbert:currentAgent` be the wildcard agent (`*`) or a runtime substitution for `odrl:assignee`? | Issue 1 | Resolved: neither -- omit assignee for universal rules, keep `currentAgent` for constraints only |
| Q2 | Keep `adalbert:rightOperandRef`, or switch to `odrl:rightOperandReference` with explicit RDF mapping? | Issue 3 | Open |
| Q3 | Keep custom `partOf`/`memberOf`, or converge on `odrl:partOf` with additional constraints? | Issue 4 | Open |

---

## Resolution Log

| Date | Issue | Action | Commit |
|------|-------|--------|--------|
| 2026-02-04 | Issue 1, Q1 | Separated concerns: `currentAgent` for constraints only (RL2-aligned); absent assignee = universal applicability; removed from 11 example rules | -- |
