# Adalbert Documentation

## Specification

| Document | Description |
|----------|-------------|
| [Adalbert_Semantics.md](Adalbert_Semantics.md) | Formal semantics (normative) |

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

Adalbert **extends** ODRL 2.2, not restricts it. Key additions:
- Duty lifecycle states (Pending/Active/Fulfilled/Violated)
- Bilateral agreement evaluation
- `adalbert:target` for policy-level assets
- `adalbert:partOf` for asset hierarchies

Adalbert policies require Adalbert-aware processors.

### 2. Bilateral Agreements

ODRL evaluates agreements from assignee perspective only. Adalbert returns duties for **both** grantor and grantee, enabling provider SLAs.

### 3. Deadline Semantics

Supports both:
- `xsd:dateTime`: absolute deadline (2026-12-31T23:59:59Z)
- `xsd:duration`: relative to activation (P30D, PT24H)

### 4. Offer Policy Type

Added `adalbert:Offer` between Set and Agreement:
- Set: no parties
- Offer: grantor required, grantee optional
- Agreement: both required

### 5. DCON Alignment

Contracts extension (`adalbert-dc:`) uses `skos:closeMatch` for DCON mappings, not `owl:sameAs`, because state machines differ.

---

## Namespaces

```
adalbert:     https://vocabulary.bigbank/adalbert/
adalbert-gov: https://vocabulary.bigbank/adalbert/governance/
adalbert-md:  https://vocabulary.bigbank/adalbert/market-data/
adalbert-du:  https://vocabulary.bigbank/adalbert/data-use/
adalbert-dc:  https://vocabulary.bigbank/adalbert/contracts/
```

See [../config/namespaces.ttl](../config/namespaces.ttl) for authoritative registry.
