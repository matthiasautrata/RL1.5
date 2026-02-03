# Adalbert: Deterministic Policy Language for Data Governance

**A formally specified policy language extending ODRL 2.2 with deterministic evaluation semantics.**

---

## What This Is

Adalbert extends ODRL 2.2 to provide:

1. **Explicit duty lifecycle**: Pending → Active → Fulfilled/Violated
2. **Bilateral agreements**: Both grantor and grantee may have duties
3. **Deterministic evaluation**: Total functions, no undefined states
4. **Formal verification target**: Amenable to Dafny, Why3, Coq

Adalbert is **specification-first**. The semantics document defines what any conformant implementation must do.

---

## Quick Start

See [examples/](examples/) for complete working policies:

- [market-data-contract.ttl](examples/market-data-contract.ttl) — DataContract, Subscription, bilateral duties
- [data-use-policy.ttl](examples/data-use-policy.ttl) — Role-based access, purpose constraints

---

## Key Differences from ODRL 2.2

| Aspect | ODRL 2.2 | Adalbert |
|--------|----------|----------|
| Duty states | Undefined | Pending → Active → Fulfilled/Violated |
| Agreement evaluation | Unilateral (assignee) | Bilateral (grantor + grantee duties) |
| Conflict resolution | Configurable | Fixed: Prohibition > Privilege |
| Evaluation order | Undefined | Deterministic left-to-right |
| Policy types | Set, Offer, Agreement | Set, Offer, Agreement (refined semantics) |

**Note**: Adalbert is an ODRL *extension*, not a pure subset. Adalbert policies require Adalbert-aware processors.

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
    grantorDuties: Set<Duty>,   // Provider obligations (SLAs)
    granteeDuties: Set<Duty>,   // Consumer obligations
    violations: Set<Duty>
}
```

### 4. Explicit Lifecycle

Duties have observable state:

```
         condition true
Pending ──────────────> Active
                         │  │
          action done    │  │  deadline passed
                         ▼  ▼
                   Fulfilled  Violated
```

---

## Repository Structure

```
Adalbert/
├── config/
│   └── namespaces.ttl               # Authoritative namespace registry
├── ontology/
│   ├── adalbert-core.ttl            # Core classes and properties
│   ├── adalbert-shacl.ttl           # Validation shapes
│   └── adalbert-prof.ttl            # DXPROF profile declaration
├── profiles/
│   ├── adalbert-governance-core.ttl # Shared governance operands
│   ├── adalbert-market-data.ttl     # Market data licensing
│   ├── adalbert-data-use.ttl        # Internal access control
│   └── adalbert-contracts.ttl       # Contract lifecycle (DCON alignment)
├── examples/
│   ├── market-data-contract.ttl     # Complete contract example
│   └── data-use-policy.ttl          # Access control example
└── docs/
    ├── Adalbert_Semantics.md        # Formal semantics (normative)
    └── comparisons/
        └── comparison-dcon.md       # DCON integration guide
```

---

## DCON Integration

Adalbert's contracts extension (`adalbert-dc:`) provides full alignment with DCON:

| DCON | Adalbert | Mapping |
|------|----------|---------|
| `dcon:DataContract` | `adalbert-dc:DataContract` | Both are Offers |
| `dcon:DataContractSubscription` | `adalbert-dc:Subscription` | Both are Agreements |
| `dcon:Promise` | `adalbert:Duty` | Partial (simplified) |
| Contract states | Contract states | `skos:closeMatch` |

See [comparison-dcon.md](docs/comparisons/comparison-dcon.md) for complete mapping with examples.

---

## Namespaces

| Prefix | Namespace |
|--------|-----------|
| `adalbert:` | `https://vocabulary.bigbank/adalbert/` |
| `adalbert-gov:` | `https://vocabulary.bigbank/adalbert/governance/` |
| `adalbert-md:` | `https://vocabulary.bigbank/adalbert/market-data/` |
| `adalbert-du:` | `https://vocabulary.bigbank/adalbert/data-use/` |
| `adalbert-dc:` | `https://vocabulary.bigbank/adalbert/contracts/` |

---

## Conformance

An implementation conforms to Adalbert if:

1. It accepts policies that validate against `adalbert-shacl.ttl`
2. Its evaluation function produces identical results for identical inputs
3. All functions are total (no undefined behavior)
4. Duty state transitions match the operational semantics
5. Agreement evaluation returns duties for both parties

---

## References

- [ODRL 2.2 Information Model](https://www.w3.org/TR/odrl-model/)
- [W3C Market Data Profile](https://www.w3.org/2021/md-odrl-profile/v1/)
- [W3C ODRL Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)

---

**Version**: 0.2 | **Date**: 2026-02-03
