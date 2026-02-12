# Adalbert: Deterministic Policy Language for Data Governance

**An ODRL 2.2 profile with deterministic evaluation semantics for data governance.**

---

## What This Is

Adalbert is a proper ODRL 2.2 profile. It uses ODRL terms for all standard constructs (Permission, Duty, Prohibition, Agreement, etc.) and only adds extensions where ODRL 2.2 leaves behavior undefined:

1. **Explicit lifecycle**: Pending → Active → Fulfilled/Violated (unified for duties and contracts)
2. **Bilateral agreements**: Both assigner and assignee may have duties
3. **Deterministic evaluation**: Total functions, no undefined states
4. **Formal verification target**: Amenable to Dafny, Why3, Coq
5. **Structured operand resolution**: `resolutionPath` from canonical roots (agent, asset, context)
6. **Recurring duties**: `recurrence` via RFC 5545 RRULE for scheduled obligations

Adalbert is **specification-first**. The semantics document defines what any conformant implementation must do. Every Adalbert policy is a valid ODRL 2.2 policy.

---

## Quick Start

See [examples/](examples/) for complete working policies:

- [data-contract.ttl](examples/data-contract.ttl) — DataContract, Subscription, bilateral duties
- [data-use-policy.ttl](examples/data-use-policy.ttl) — Role-based access, purpose constraints

---

## What Adalbert Adds to ODRL 2.2

| Extension | ODRL 2.2 | What Adalbert Adds |
|-----------|----------|--------------------|
| Duty lifecycle | Undefined | Pending → Active → Fulfilled/Violated |
| Bilateral duties | Unilateral (assignee only) | Assigner duties + assignee duties |
| Conflict resolution | Configurable | Fixed: Prohibition > Permission |
| Evaluation order | Undefined | Deterministic left-to-right |
| Operand resolution | Implicit | Explicit `resolutionPath` from canonical roots |
| Recurring duties | — | `recurrence` via RFC 5545 RRULE with per-instance `deadline` |
| Contract types | — | `DataContract` (subclass of Offer), `Subscription` (subclass of Agreement) |

**Note**: Adalbert is an ODRL profile, not a parallel vocabulary. Standard ODRL processors can parse Adalbert policies; Adalbert-aware processors additionally enforce lifecycle, bilateral duties, and deterministic evaluation.

---

## Design Principles

### 1. Specification Precedes Implementation

The [formal semantics](docs/Adalbert_Semantics.md) is the normative reference.

### 2. Total Functions

Every evaluation terminates with a defined result:

```
Eval : Request × PolicySet × State → Decision × DutySet
```

### 3. Bilateral Agreements

Agreements return duties for **both** parties:

```
Result = {
    decision: Permit | Deny | NotApplicable,
    assignerDuties: Set<Duty>,   // Provider obligations (SLAs)
    assigneeDuties: Set<Duty>,   // Consumer obligations
    violations: Set<Duty>
}
```

### 4. Unified Lifecycle State

Duties and contracts share four states:

```
         condition true
Pending ──────────────> Active
                         │  │
          action done    │  │  deadline passed
                         ▼  ▼
                   Fulfilled  Violated
```

An `odrl:Duty` progresses through `adalbert:State` values. An `adalbert:DataContract` shares the same state machine.

### 5. Structured Operand Resolution

Operands resolve via `resolutionPath` — dot-separated paths from canonical roots:

```
agent.role, agent.organization, agent.costCenter
asset.classification, asset.market, asset.isBenchmark
context.purpose, context.environment, context.legalBasis
```

---

## Repository Structure

```
Adalbert/
├── config/
│   └── namespaces.ttl               # Authoritative namespace registry
├── ontology/
│   ├── adalbert-core.ttl            # ODRL profile extension
│   ├── adalbert-shacl.ttl           # Validation shapes
│   ├── adalbert-prof.ttl            # DXPROF profile declaration
├── profiles/
│   └── adalbert-due.ttl             # Data use vocabulary (all operands + actions)
├── examples/
│   ├── data-contract.ttl            # Complete contract example (with recurrence)
│   ├── data-use-policy.ttl          # Access control example
│   └── baseline.ttl                 # Comprehensive test data (8 contracts, 2 subscriptions)
└── docs/
    ├── adalbert-overview.md         # What is Adalbert? (start here)
    ├── adalbert-specification.md    # Technical vocabulary reference
    ├── adalbert-term-mapping.md     # Business term -> property mapping + DCON migration
    ├── Adalbert_Semantics.md        # Formal semantics (normative)
    ├── contracts-guide.md           # Data contract authoring guide
    ├── policy-writers-guide.md      # Data use policy authoring guide
    └── comparisons/
        └── comparison-dcon.md       # DCON supersession analysis
```

---

## DCON Supersession

As of v0.7, Adalbert **supersedes** DCON. DCON's promise hierarchy dissolves into standard `odrl:Duty` patterns with DUE actions (`deliver`, `notify`, `conformTo`, `report`). Recurring obligations use `adalbert:recurrence` (RFC 5545 RRULE) instead of DCON's scheduling constraints.

| DCON | Adalbert v0.7 | Status |
|------|---------------|--------|
| `dcon:DataContract` | `adalbert:DataContract` | Absorbed into core |
| `dcon:DataContractSubscription` | `adalbert:Subscription` | Absorbed into core |
| `dcon:Promise` hierarchy | `odrl:Duty` + DUE actions | Dissolved |
| `dcon:promisedDeliveryTime` | `adalbert:recurrence` + `adalbert:deadline` | Scheduling + window |

See [comparison-dcon.md](docs/comparisons/comparison-dcon.md) for the supersession analysis, [adalbert-term-mapping.md](docs/adalbert-term-mapping.md) for complete DCON -> Adalbert property mapping, and [contracts-guide.md](docs/contracts-guide.md) for the contracts authoring guide.

> **Promise terminology.** If you prefer DCON-style "promise" naming (ProviderPromise, QualityPromise, etc.), this can be reintroduced as syntactic sugar. A promise is a Duty where `adalbert:subject` equals the policy's `odrl:assigner`. Two options: **(1)** LinkML authoring sugar — a `promise` class that expands to a standard Duty, no ontology change; **(2)** OWL thin alias — `adalbert:Promise rdfs:subClassOf odrl:Duty` with a SHACL constraint, making promises queryable in SPARQL. See [issues.md Q1](docs/issues.md#q1--promise-as-syntactic-sugar-over-duty) for details.

---

## Namespaces

| Prefix | Namespace | Role |
|--------|-----------|------|
| `odrl:` | `http://www.w3.org/ns/odrl/2/` | Primary — all standard constructs |
| `adalbert:` | `https://vocabulary.bigbank/adalbert/` | Extensions only (State, deadline, recurrence, DataContract, Subscription, resolutionPath, hierarchy) |
| `adalbert-due:` | `https://vocabulary.bigbank/adalbert/due/` | Data use vocabulary (operands, domain-specific actions, concept values); ODRL Common Vocabulary actions used directly |

---

## Conformance

An implementation conforms to Adalbert if:

1. It accepts policies that validate against `adalbert-shacl.ttl`
2. It uses `odrl:Permission`, `odrl:Duty`, `odrl:Prohibition`, `odrl:Agreement` for standard constructs
3. Its evaluation function produces identical results for identical inputs
4. All functions are total (no undefined behavior)
5. State transitions match the operational semantics
6. Agreement evaluation returns duties for both assigner and assignee

---

## References

- [ODRL 2.2 Information Model](https://www.w3.org/TR/odrl-model/)
- [W3C Market Data Profile](https://www.w3.org/2021/md-odrl-profile/v1/)
- [W3C ODRL Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)

---

**Version**: 0.7 | **Date**: 2026-02-04
