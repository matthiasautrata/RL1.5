# LLM.md

Shared context and instructions for LLMs working on Adalbert.

---

## Project Overview

Adalbert is a **deterministic rights language** -- a formally specified, verification-ready ODRL 2.2 profile designed for data governance. It fixes ODRL 2.2's ambiguities (duty lifecycle, pre-conditions vs obligations, evaluation determinism) while remaining a proper profile: every Adalbert policy is a valid ODRL 2.2 policy.

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
         │ proper profile (thin extension)
         ▼
       Adalbert ──────────────> RL2 (Full Hohfeldian Framework)
         │                        Claims, Powers, Promises
         │ profile
         │
         ▼
        DUE               Semantipolis     Themis
      (vocab)           (Governance Plane)  (Runtime)
```

Key relationships:

| Project | Relationship to Adalbert |
|---------|-------------------------|
| **RL2** | Adalbert is a strict subset; upgrade by adding capabilities |
| **ODRL 2.2** | Adalbert is a proper profile; uses ODRL classes/properties directly |
| **DCON** | Alignment via skos:closeMatch; DataContract/Subscription in core |
| **Semantipolis** | Uses Adalbert for governance evaluation in the semantic layer |
| **Themis** | Existing ODRL runtime; backward-compatibility target |

---

## Key Files

| File | Purpose |
|------|---------|
| `AGENTS.md` | **Governance, personas, multi-agent coordination** |
| `docs/Adalbert_Semantics.md` | **Formal operational semantics (normative)** |
| `README.md` | Project overview, design principles, repository structure |
| `ontology/adalbert-core.ttl` | **OWL ontology -- thin profile extension: State, deadline, DataContract, Subscription, partOf, memberOf, resolutionPath, RuntimeReference, not** |
| `ontology/adalbert-shacl.ttl` | **SHACL validation shapes** |
| `ontology/adalbert-prof.ttl` | W3C DXPROF profile metadata |
| `ontology/adalbert-dcon-alignment.ttl` | DCON alignment mappings (skos:closeMatch) |
| `profiles/README.md` | Profile architecture and extension rules |
| `profiles/adalbert-due.ttl` | Data use vocabulary (all operands, actions, concept values) |
| `docs/comparisons/comparison-odrl22.md` | Detailed ODRL 2.2 comparison |
| `docs/comparisons/comparison-dcon.md` | DCON alignment analysis |
| `docs/comparisons/comparison-odrl-profiles.md` | W3C ecosystem positioning |
| `docs/comparisons/comparison-w3c-market-data.md` | W3C Market Data Profile mapping |
| `docs/conformance-w3c-best-practices.md` | W3C Best Practices compliance |
| `docs/comparisons/namespace-alignment.md` | Namespace coordination with DCON2 |

---

## Architecture

### ODRL Profile Design

Adalbert is a proper ODRL 2.2 profile. It uses ODRL's classes and properties directly and only adds terms for genuinely new capabilities.

**From ODRL (used directly):**

| ODRL Term | Role in Adalbert |
|-----------|-----------------|
| `odrl:Permission` | Permission rule (grant access) |
| `odrl:Duty` | Duty rule (required action) |
| `odrl:Prohibition` | Prohibition rule (deny access) |
| `odrl:Policy` | Abstract policy base |
| `odrl:Set` | Policy with clauses, no parties |
| `odrl:Offer` | Policy with assigner |
| `odrl:Agreement` | Policy with assigner and assignee |
| `odrl:Constraint` | Atomic constraint (leftOperand, operator, rightOperand) |
| `odrl:LogicalConstraint` | And/Or compound constraint |
| `odrl:Party` | Agent (assigner, assignee) |
| `odrl:Asset` | Target of a rule |
| `odrl:Action` | Action type |
| `odrl:LeftOperand` | Left operand of a constraint |
| `odrl:assignee` | Subject of a rule (who receives the right/duty) |
| `odrl:assigner` | Grantor of a policy (who grants rights) |
| `odrl:action` | Action of a rule |
| `odrl:target` | Object/asset of a rule |
| `odrl:constraint` | Constraint on a rule |
| `odrl:permission` | Permission clause of a policy |
| `odrl:prohibition` | Prohibition clause of a policy |
| `odrl:obligation` | Obligation clause of a policy |
| `odrl:leftOperand` | Left operand of a constraint |
| `odrl:operator` | Comparison operator of a constraint |
| `odrl:rightOperand` | Right operand of a constraint |
| `odrl:includedIn` | Action subsumption hierarchy |

**Added by Adalbert (genuinely new):**

| Adalbert Term | Purpose |
|---------------|---------|
| `adalbert:State` | Unified lifecycle state (Pending, Active, Fulfilled, Violated) |
| `adalbert:state` | Current lifecycle state property |
| `adalbert:deadline` | Time constraint for duty fulfillment |
| `adalbert:DataContract` | `rdfs:subClassOf odrl:Offer` -- data access offer |
| `adalbert:Subscription` | `rdfs:subClassOf odrl:Agreement` -- activated contract |
| `adalbert:subscribesTo` | Links Subscription to its DataContract |
| `adalbert:effectiveDate` | When contract/subscription takes effect |
| `adalbert:expirationDate` | When contract/subscription expires |
| `adalbert:partOf` | Asset hierarchy (transitive) |
| `adalbert:memberOf` | Party hierarchy (transitive) |
| `adalbert:resolutionPath` | Dot-separated path from canonical root to value |
| `adalbert:RuntimeReference` | Value resolved at evaluation time |
| `adalbert:currentAgent` | Runtime reference: the requesting agent |
| `adalbert:currentDateTime` | Runtime reference: evaluation timestamp |
| `adalbert:rightOperandRef` | Runtime reference for right-operand comparison |
| `adalbert:not` | Logical negation on LogicalConstraint (ODRL lacks this) |

### Core Abstract Syntax

```
Rule ::= Permission(assignee, action, target, constraint?)
       | Duty(assignee, action, target, constraint?, deadline?)
       | Prohibition(assignee, action, target, constraint?)

Policy ::= Set(permission | prohibition | obligation)+
         | Offer(assigner, permission | prohibition | obligation)+
         | Agreement(assigner, assignee, permission | prohibition | obligation)+

Constraint ::= odrl:Constraint(leftOperand, operator, rightOperand)
             | odrl:LogicalConstraint(And(Constraint+) | Or(Constraint+) | Not(Constraint))
```

### Unified Lifecycle State

```
         constraint becomes true
Pending ────────────────────────> Active

          action performed
Active  ────────────────────────> Fulfilled

         deadline ∧ ¬performed
Active  ────────────────────────> Violated
```

Four states apply to both duties and contracts:
- **Pending**: constraint not yet met / not yet in force
- **Active**: constraint met, action required / in force
- **Fulfilled**: action performed / obligations complete
- **Violated**: deadline passed / breached

### Operand Resolution

Operands are `odrl:LeftOperand` instances. Resolution uses `adalbert:resolutionPath` -- a dot-separated path from a canonical root:

| Root | Meaning | Examples |
|------|---------|----------|
| `agent` | Requesting agent | `agent.role`, `agent.organization` |
| `asset` | Target asset | `asset.classification`, `asset.market` |
| `context` | Request context | `context.purpose`, `context.environment` |

### Conflict Resolution

Fixed precedence: **Prohibition > Permission**. No configurable conflict strategies.

### Profile Architecture

```
          adalbert-core.ttl
          (thin profile extension:
           State, deadline,
           DataContract, Subscription,
           partOf, memberOf,
           resolutionPath,
           RuntimeReference, not)
                 │
                 ▼
            adalbert-due
            (all vocab:
             purpose, role,
             env, actions)
```

One profile (DUE) carries all vocabulary. Contract lifecycle classes (DataContract, Subscription) live in core. DCON alignment is a separate mapping file, not a profile.

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
| **ODRL 2.2** | Base standard; Adalbert is a proper profile | Classes, properties used directly |
| **OWL 2** | Profile extension, class/property definitions | `ontology/adalbert-core.ttl` |
| **SHACL** | Validation shapes for policy conformance | `ontology/adalbert-shacl.ttl` |
| **SKOS** | Concept hierarchies for operand values | Profile vocabularies |
| **DXPROF** | W3C Profiles Vocabulary for metadata | `ontology/adalbert-prof.ttl` |
| **DCON** | Data Contract Ontology alignment | `ontology/adalbert-dcon-alignment.ttl` |
| **Dublin Core** | Metadata (title, description, date) | Throughout RDF files |

### URI Namespace

```
adalbert:      https://vocabulary.bigbank/adalbert/
adalbert-due:  https://vocabulary.bigbank/adalbert/due/
```

---

## Common Tasks

### Add a new left operand

1. Add to `profiles/adalbert-due.ttl`
2. Check existing operands for overlap
3. Define as an `odrl:LeftOperand` with `adalbert:resolutionPath` (must start with `agent.`, `asset.`, or `context.`), and SKOS hierarchy if needed
4. If the operand needs SHACL validation, add shape constraints
5. Update `profiles/README.md`
6. Review with Ontologist persona

### Add a new action

1. Define in `profiles/adalbert-due.ttl` with `rdf:type odrl:Action`
2. Declare `odrl:includedIn` hierarchy (subsumption)
3. Verify it doesn't conflict with existing actions
4. Update profile documentation

### Add a new domain profile

1. Create `profiles/adalbert-<domain>.ttl`
2. Import `adalbert:` (core)
3. Define domain-specific operands and actions
4. Add to `ontology/adalbert-prof.ttl` as a DXPROF resource descriptor
5. Write a comparison document in `docs/comparisons/` if aligning to an external standard
6. Review: Ontologist persona for vocabulary, Reviewer persona for consistency

### Modify the formal semantics

1. This is a **human-decides** change -- present rationale first
2. Draft changes in `docs/Adalbert_Semantics.md`
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
3. **Prohibition > Permission** -- fixed conflict resolution, no ambiguity
4. **Strict subset of RL2** -- every Adalbert policy is a valid RL2 policy; upgrade without migration
5. **Proper ODRL 2.2 profile** -- uses ODRL classes and properties directly; adds only genuinely new terms
6. **Profiles extend vocabulary, not semantics** -- profiles add operands and actions, never new rule or policy types
7. **Backward compatible with ODRL 2.2** -- all Adalbert policies are valid ODRL policies; any ODRL processor can partially understand them

---

## Open Questions

| Question | Context | Status |
|----------|---------|--------|
| Final namespace URIs | Currently using `vocabulary.bigbank` | Open |
| Dafny mechanization strategy | Primary verification target | Phase 2 |
| Go extraction from Dafny | Reference implementation language | Phase 2 |
| Themis backward compatibility | How much of existing runtime to preserve | Open |
| Profile SHACL shapes | Per-profile validation beyond core shapes | Incomplete |
| DCON 1.0 compatibility verification | Formal mapping needed | Open |
| ABAC patterns for data-use profile | Attribute-based access control | Planned |

---

## Status

- **Version:** 0.5 (Draft)
- **Phase:** 1 -- Specification
- **Formal semantics:** Complete draft (`docs/Adalbert_Semantics.md`)
- **OWL ontology:** Complete draft (`ontology/adalbert-core.ttl`)
- **SHACL shapes:** Complete draft (`ontology/adalbert-shacl.ttl`)
- **Profiles:** DUE (data use vocabulary)
- **DCON alignment:** Mapping file (`ontology/adalbert-dcon-alignment.ttl`)
- **Comparison docs:** Complete for ODRL 2.2, W3C profiles, market data, DCON
- **Next:** Profile review, SHACL completeness, namespace finalization, Phase 2 planning
