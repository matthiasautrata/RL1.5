# LLM.md

Shared context and instructions for LLMs working on Adalbert.

---

## Project Overview

Adalbert is a **deterministic rights language** -- a formally specified, verification-ready subset of RL2 designed for data governance. It fixes ODRL 2.2's ambiguities (duty lifecycle, pre-conditions vs obligations, evaluation determinism) while remaining backward-compatible: every Adalbert policy is a valid ODRL 2.2 policy.

The project is **specification-first**. The formal semantics document is normative. Implementations derive from the specification, not the other way around.

```
Eval : Request × PolicySet × State → Decision × DutySet
```

This function is total. Every evaluation terminates with a defined result. No undefined states. No implicit failures.

**See also:** `AGENTS.md` for governance, personas, and decision authority.

---

## Lineage

Adalbert sits in a broader ecosystem of formal governance specifications:

```
    ODRL 2.2 (W3C Standard)
         │ semantic clarification, strict subset
         ▼
       Adalbert ──────────────> RL2 (Full Hohfeldian Framework)
         │                        Claims, Powers, Promises
         │ domain profiles
    ┌────┼────────┐
    │    │        │
    ▼    ▼        ▼
 Market  Data   DCON         Semantipolis     Themis
  Data    Use  Conform.   (Governance Plane)  (Runtime)
```

Key relationships:

| Project | Relationship to Adalbert |
|---------|-----------------------|
| **RL2** | Adalbert is a strict subset; upgrade by adding capabilities |
| **ODRL 2.2** | Adalbert fixes semantic gaps; all Adalbert policies are valid ODRL |
| **DCON** | DCON contracts contain Adalbert policies; Promise deferred to RL2 |
| **Semantipolis** | Uses Adalbert for governance evaluation in the semantic layer |
| **Themis** | Existing ODRL runtime; backward-compatibility target |

---

## Key Files

| File | Purpose |
|------|---------|
| `AGENTS.md` | **Governance, personas, multi-agent coordination** |
| `docs/RL1_5_Semantics.md` | **Formal operational semantics (normative)** |
| `README.md` | Project overview, design principles, repository structure |
| `ontology/adalbert-core.ttl` | **OWL ontology -- core classes and properties** |
| `ontology/adalbert-shacl.ttl` | **SHACL validation shapes** |
| `ontology/adalbert-prof.ttl` | W3C DXPROF profile metadata |
| `profiles/README.md` | Profile architecture and extension rules |
| `profiles/adalbert-governance-core.ttl` | Shared governance operands and actions |
| `profiles/adalbert-market-data.ttl` | Market data licensing vocabulary |
| `profiles/adalbert-data-use.ttl` | Internal access control vocabulary |
| `profiles/adalbert-dcon.ttl` | DCON conformance declaration |
| `docs/comparisons/comparison-odrl22.md` | Detailed ODRL 2.2 comparison |
| `docs/comparisons/comparison-dcon.md` | DCON alignment analysis |
| `docs/comparisons/comparison-odrl-profiles.md` | W3C ecosystem positioning |
| `docs/comparisons/comparison-w3c-market-data.md` | W3C Market Data Profile mapping |
| `docs/conformance-w3c-best-practices.md` | W3C Best Practices compliance |
| `docs/comparisons/namespace-alignment.md` | Namespace coordination with DCON2 |

---

## Architecture

### Core Normative Model

```
Norm ::= Privilege(subject, action, asset, condition?)
       | Duty(subject, action, asset, condition?, deadline?)
       | Prohibition(subject, action, asset, condition?)

Policy ::= Set(clause+)
         | Agreement(grantor, grantee, clause+)

Condition ::= AtomicConstraint(leftOperand, operator, rightOperand)
            | And(Condition+) | Or(Condition+) | Not(Condition)
```

### Duty Lifecycle (State Machine)

```
         condition becomes true
Pending ────────────────────────> Active

          action performed
Active  ────────────────────────> Fulfilled

         deadline ∧ ¬performed
Active  ────────────────────────> Violated
```

### Conflict Resolution

Fixed precedence: **Prohibition > Privilege**. No configurable conflict strategies.

### Profile Architecture

```
                adalbert-core.ttl
                (Norm, Condition, Policy)
                       │
                       ▼
          adalbert-governance-core.ttl
          (Shared: purpose, classification,
           jurisdiction, retention, audit)
                       │
     ┌─────────────────┼─────────────────┐
     │                 │                 │
     ▼                 ▼                 ▼
  market-data       data-use           dcon
  (display, seats,  (role, dept,      (Promise ext,
   redistribution)   environment)      contract status)
```

### What Adalbert Defers to RL2

| Concept | Reason |
|---------|--------|
| Claim, Power, Liability, Immunity | Second-order norms |
| Promise Theory | Voluntary cooperation model |
| EventConstraint | Operational complexity |
| Policy Generations | Versioning layer |
| Identity Binding (Tun-sollen) | Advanced pattern |
| Xone operator | Rarely needed |

---

## Standards

| Standard | Used For | Where |
|----------|----------|-------|
| **ODRL 2.2** | Base standard; Adalbert is a semantic clarification | Alignment throughout |
| **OWL 2** | Core ontology, class/property definitions | `ontology/adalbert-core.ttl` |
| **SHACL** | Validation shapes for policy conformance | `ontology/adalbert-shacl.ttl` |
| **SKOS** | Concept hierarchies for operand values | Profile vocabularies |
| **DXPROF** | W3C Profiles Vocabulary for metadata | `ontology/adalbert-prof.ttl` |
| **DCAT** | Dataset/catalog references | Via DCON integration |
| **DCON** | Data Contract Ontology | `profiles/adalbert-dcon.ttl` |
| **Dublin Core** | Metadata (title, description, date) | Throughout RDF files |

### URI Namespace

```
adalbert:      https://adalbert.example/
rl15gov:   https://adalbert.example/profiles/governance#
rl15mkt:   https://adalbert.example/profiles/market-data#
rl15du:    https://adalbert.example/profiles/data-use#
```

Note: Using `example` URIs during draft phase. Final namespace TBD.

---

## Common Tasks

### Add a new left operand to a profile

1. Determine which profile it belongs to (governance-core, market-data, data-use)
2. Check `adalbert-governance-core.ttl` for existing operands that may already cover it
3. Define the operand with `contextKey`, `supportedOperators`, and SKOS hierarchy if needed
4. If the operand needs SHACL validation, add shape constraints
5. Update `profiles/README.md` operand table
6. Review with Ontologist persona

### Add a new action

1. Define in the appropriate profile with `rdf:type adalbert:Action`
2. Declare `includedIn` hierarchy (subsumption)
3. Verify it doesn't conflict with existing actions
4. Update profile documentation

### Add a new domain profile

1. Create `profiles/adalbert-<domain>.ttl`
2. Import `adalbert-governance-core.ttl`
3. Define domain-specific operands and actions
4. Add to `ontology/adalbert-prof.ttl` as a DXPROF resource descriptor
5. Write a comparison document in `docs/comparisons/` if aligning to an external standard
6. Review: Ontologist persona for vocabulary, Reviewer persona for consistency

### Modify the formal semantics

1. This is a **human-decides** change -- present rationale first
2. Draft changes in `docs/RL1_5_Semantics.md`
3. Verify the OWL ontology and SHACL shapes still align
4. Check that all four correctness theorems (Totality, Determinism, Monotonicity, Duty Progress) still hold
5. Update comparison documents if the change affects ODRL 2.2 alignment
6. Review: Architect persona for semantics, Reviewer persona for cross-document consistency

### Write a comparison document

1. Identify the external standard and its normative references
2. Analyze alignment dimension by dimension
3. Use tables for systematic comparison, prose for nuance
4. Distinguish: what Adalbert matches, what it clarifies, what it defers
5. Review: Editor persona for clarity, Ontologist persona for technical accuracy

---

## Principles

1. **Specification is normative** -- the formal semantics document defines correctness, not any implementation
2. **Total functions** -- every evaluation terminates with a defined result
3. **Prohibition > Privilege** -- fixed conflict resolution, no ambiguity
4. **Strict subset of RL2** -- every Adalbert policy is a valid RL2 policy; upgrade without migration
5. **Standards over invention** -- build on ODRL 2.2, SHACL, OWL, SKOS, DXPROF
6. **Profiles extend vocabulary, not semantics** -- profiles add operands and actions, never new norm or policy types
7. **Backward compatible with ODRL 2.2** -- all Adalbert policies are valid ODRL policies

---

## Open Questions

| Question | Context | Status |
|----------|---------|--------|
| Final namespace URIs | Currently using `adalbert.example` | Open |
| Dafny mechanization strategy | Primary verification target | Phase 2 |
| Go extraction from Dafny | Reference implementation language | Phase 2 |
| Themis backward compatibility | How much of existing runtime to preserve | Open |
| Profile SHACL shapes | Per-profile validation beyond core shapes | Incomplete |
| DCON 1.0 compatibility verification | Formal mapping needed | Open |
| Exchange-specific market data constraints | Bloomberg, Reuters specifics | Planned |
| ABAC patterns for data-use profile | Attribute-based access control | Planned |

---

## Status

- **Version:** 0.1 (Draft)
- **Phase:** 1 -- Specification
- **Formal semantics:** Complete draft (`docs/RL1_5_Semantics.md`)
- **OWL ontology:** Complete draft (`ontology/adalbert-core.ttl`)
- **SHACL shapes:** Complete draft (`ontology/adalbert-shacl.ttl`)
- **Profiles:** Draft 0.1 (governance-core, market-data, data-use, dcon)
- **Comparison docs:** Complete for ODRL 2.2, W3C profiles, market data, DCON
- **Next:** Profile review, SHACL completeness, namespace finalization, Phase 2 planning
