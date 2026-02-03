# RL1.5: Deterministic Policy Language for Data Governance

**A formally specified policy language extending ODRL 2.2 with deterministic evaluation semantics.**

---

## What This Is

RL1.5 extends ODRL 2.2 to provide:

1. **Explicit duty lifecycle**: Pending → Active → Fulfilled/Violated
2. **Bilateral agreements**: Both grantor and grantee may have duties
3. **Deterministic evaluation**: Total functions, no undefined states
4. **Formal verification target**: Amenable to Dafny, Why3, Coq

RL1.5 is **specification-first**. The semantics document defines what any conformant implementation must do.

---

## Key Differences from ODRL 2.2

| Aspect | ODRL 2.2 | RL1.5 |
|--------|----------|-------|
| Duty states | Undefined | Pending → Active → Fulfilled/Violated |
| Agreement evaluation | Unilateral (assignee) | Bilateral (grantor + grantee duties) |
| Conflict resolution | Configurable | Fixed: Prohibition > Privilege |
| Evaluation order | Undefined | Deterministic left-to-right |
| Policy types | Set, Offer, Agreement | Set, Offer, Agreement (with refined semantics) |

**Note**: RL1.5 is an ODRL *extension*, not a pure subset. RL1.5 policies require RL1.5-aware processors.

---

## Design Principles

### 1. Specification Precedes Implementation

The [formal semantics](docs/RL1_5_Semantics.md) is the normative reference.

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
    grantorDuties: Set<Duty>,   // Provider obligations
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
RL1.5/
├── README.md
├── config/
│   └── namespaces.ttl               # Authoritative namespace registry
├── ontology/
│   ├── rl1_5-core.ttl               # Core classes and properties
│   └── rl1_5-shacl.ttl              # Validation shapes
├── profiles/
│   ├── rl1_5-governance-core.ttl    # Shared governance operands
│   ├── rl1_5-market-data.ttl        # Market data licensing
│   ├── rl1_5-data-use.ttl           # Internal access control
│   └── rl1_5-contracts.ttl          # Contract lifecycle (DCON alignment)
└── docs/
    ├── RL1_5_Semantics.md           # Formal semantics (normative)
    └── comparisons/                 # ODRL, W3C, DCON comparisons
```

---

## Namespaces

| Prefix | Namespace | Description |
|--------|-----------|-------------|
| `rl15:` | `https://vocabulary.bigbank/rl1.5/` | Core vocabulary |
| `rl15-gov:` | `https://vocabulary.bigbank/rl1.5/governance/` | Governance operands |
| `rl15-md:` | `https://vocabulary.bigbank/rl1.5/market-data/` | Market data profile |
| `rl15-du:` | `https://vocabulary.bigbank/rl1.5/data-use/` | Data use profile |
| `rl15-dc:` | `https://vocabulary.bigbank/rl1.5/contracts/` | Contracts extension |

---

## Conformance

An implementation conforms to RL1.5 if:

1. It accepts policies that validate against `rl1_5-shacl.ttl`
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

## Status

**Version**: 0.2  
**Date**: 2026-02-03
