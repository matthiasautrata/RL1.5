# RL1.5: Deterministic Rights Language

**A formally specified, verification-ready subset of RL2 for data governance.**

---

## What This Is

RL1.5 is a **strict semantic subset** of RL2 designed to:

1. Fix ODRL 2.2's ambiguities (duty lifecycle, pre-conditions vs obligations)
2. Provide deterministic, total evaluation semantics
3. Be amenable to future formal verification (Dafny, Why3, Coq)
4. Serve as the semantic foundation for data governance across domains

RL1.5 is **specification-first**. The semantics document defines what any conformant implementation must do. Implementations are derived from the specification, not the other way around.

---

## Design Principles

### 1. Specification Precedes Implementation

The formal semantics (RL1_5_Semantics.md) is the normative reference. Any implementation that produces different results is incorrect.

### 2. Total Functions

Every evaluation terminates with a defined result. No "undefined" states, no implicit failures. The evaluation function:

```
Eval : Request × PolicySet × State → Decision × DutySet
```

...always returns. Malformed inputs produce explicit error values, not exceptions.

### 3. Small-Step Operational Semantics

State transitions are explicit and enumerable:

```
(Σ, e) → (Σ', e')
```

This structure supports:
- Formal verification
- Audit trail reconstruction
- Debugging (single-step evaluation)

### 4. Strict Subset of RL2

Every RL1.5 policy is a valid RL2 policy. Migration to full RL2 requires no policy changes—only adding capabilities (Claims, Powers, Promises).

---

## What RL1.5 Contains

### Normative Layer

```
Norm ::= Privilege(subject, action, asset, condition?)
       | Duty(subject, action, asset, condition?, deadline?)
       | Prohibition(subject, action, asset, condition?)
```

### Duty Lifecycle

```
         condition becomes true
Pending ────────────────────────> Active

          action performed
Active  ────────────────────────> Fulfilled

         deadline ∧ ¬performed
Active  ────────────────────────> Violated
```

### Conditions

```
Condition ::= AtomicConstraint(leftOperand, operator, rightOperand)
            | And(Condition+)
            | Or(Condition+)
            | Not(Condition)
```

### Policies

```
Policy ::= Set(clause+)
         | Agreement(grantor, grantee, clause+)
```

---

## What RL1.5 Defers to RL2

| Concept | Why Deferred |
|---------|--------------|
| Claim, Power, Liability, Immunity | Second-order norms (norms about norms) |
| Promise Theory | Voluntary cooperation model |
| EventConstraint | Operational complexity |
| Policy Generations | Versioning layer |
| Identity Binding (Tun-sollen) | Advanced pattern |
| Xone operator | Rarely needed |

These can be added by upgrading to RL2 without changing existing policies.

---

## Repository Structure

```
RL1.5/
├── README.md                    # This file
├── RL1_5_Semantics.md           # Formal semantics (normative)
├── RL1_5_Vocabulary.md          # Class/property reference
├── ontology/
│   ├── rl1_5-core.ttl           # OWL ontology
│   └── rl1_5-shacl.ttl          # SHACL validation shapes
└── profiles/
    ├── rl1_5-governance-core.ttl    # Shared data governance operands
    ├── rl1_5-market-data.ttl        # Market data licensing
    ├── rl1_5-data-use.ttl           # Internal access control
    └── rl1_5-dcon.ttl               # DCON conformance declaration
```

---

## Relationship to Other Projects

```
                    RL2 (Full Specification)
                           ↑
                           │ strict subset
                           │
    ┌──────────────────────┼──────────────────────┐
    │                   RL1.5                      │
    │         (Deterministic Subset)              │
    └──────────────────────┬──────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
      Market Data     Data Use         DCON
       Profile        Profile        Profile
           │               │               │
           └───────────────┼───────────────┘
                           │
                           ▼
                    Semantipolis
                  (Governance Plane)
                           │
                           ▼
                       Themis
                  (Runtime Implementation)
```

**RL2**: Full formal specification with Hohfeldian framework, Promise Theory
**RL1.5**: Verification-ready subset for practical data governance  
**Profiles**: Domain-specific operands and actions  
**Semantipolis**: Semantic control plane using RL1.5 for governance  
**Themis**: Existing Python runtime (to be upgraded or replaced)

---

## Implementation Strategy

### Phase 1: Specification (Current)

- Complete RL1_5_Semantics.md
- Define ontology and SHACL shapes
- Create domain profiles

### Phase 2: Reference Implementation

- Dafny specification with proofs
- Extract to Go (or similar)
- Property-based testing against specification

### Phase 3: Production Runtime

- High-performance evaluator
- Integration with Semantipolis
- Backward compatibility with Themis policies

---

## Conformance

An implementation conforms to RL1.5 if:

1. It accepts all policies that validate against `rl1_5-shacl.ttl`
2. Its evaluation function produces identical `(Decision, DutySet)` for identical `(Request, PolicySet, State)`
3. All functions are total (no undefined behavior)
4. State transitions match the operational semantics

---

## References

- [RL2 Specification](../RL2/README.md) - Full language specification
- [DCON Ontology](../dcon/dcon/ontology/dcon.ttl) - Data contract vocabulary
- [Semantipolis](../dcon/semantipolis/README.md) - Semantic control plane
- [Themis](../themis/README.md) - Existing ODRL runtime

---

## Status

**Version**: 0.1 (Draft)  
**Date**: 2026-02-02  
**Authors**: Data Governance Architecture Team

---

*RL1.5 is a specification. Implementations derive from it, not the other way around.*
