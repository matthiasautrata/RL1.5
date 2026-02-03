# RL1.5 Documentation

## Specification

| Document | Description |
|----------|-------------|
| [RL1_5_Semantics.md](RL1_5_Semantics.md) | Formal semantics (normative) |

## Conformance

| Document | Description |
|----------|-------------|
| [conformance-w3c-best-practices.md](conformance-w3c-best-practices.md) | W3C ODRL Profile Best Practices |

## Comparisons

| Document | Description |
|----------|-------------|
| [comparison-odrl22.md](comparisons/comparison-odrl22.md) | vs ODRL 2.2 core |
| [comparison-odrl-profiles.md](comparisons/comparison-odrl-profiles.md) | vs W3C ODRL profile ecosystem |
| [comparison-w3c-market-data.md](comparisons/comparison-w3c-market-data.md) | vs W3C Market Data Profile (+ migration appendix) |
| [comparison-dcon.md](comparisons/comparison-dcon.md) | vs DCON ontology |
| [namespace-alignment.md](comparisons/namespace-alignment.md) | Namespace coordination |

---

## Key Design Decisions (v0.2)

### 1. ODRL Extension, Not Subset

RL1.5 **extends** ODRL 2.2, not restricts it. Key additions:
- Duty lifecycle states (Pending/Active/Fulfilled/Violated)
- Bilateral agreement evaluation
- `rl15:target` for policy-level assets
- `rl15:partOf` for asset hierarchies

RL1.5 policies require RL1.5-aware processors.

### 2. Bilateral Agreements

ODRL evaluates agreements from assignee perspective only. RL1.5 returns duties for **both** grantor and grantee, enabling provider SLAs.

### 3. Deadline Semantics

Supports both:
- `xsd:dateTime`: absolute deadline (2026-12-31T23:59:59Z)
- `xsd:duration`: relative to activation (P30D, PT24H)

### 4. Offer Policy Type

Added `rl15:Offer` between Set and Agreement:
- Set: no parties
- Offer: grantor required, grantee optional
- Agreement: both required

### 5. DCON Alignment

Contracts extension (`rl15-dc:`) uses `skos:closeMatch` for DCON mappings, not `owl:sameAs`, because state machines differ.

---

## Namespaces

```
rl15:     https://vocabulary.bigbank/rl1.5/
rl15-gov: https://vocabulary.bigbank/rl1.5/governance/
rl15-md:  https://vocabulary.bigbank/rl1.5/market-data/
rl15-du:  https://vocabulary.bigbank/rl1.5/data-use/
rl15-dc:  https://vocabulary.bigbank/rl1.5/contracts/
```

See [../config/namespaces.ttl](../config/namespaces.ttl) for authoritative registry.
