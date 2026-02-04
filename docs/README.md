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

## Key Design Decisions (v0.4)

### 1. Proper ODRL 2.2 Profile

Adalbert is a proper **ODRL 2.2 profile**. It uses ODRL terms for all standard constructs (`odrl:Permission`, `odrl:Duty`, `odrl:Prohibition`, `odrl:target`, `odrl:Set`, `odrl:Offer`, `odrl:Agreement`, `odrl:LeftOperand`, etc.) and only introduces extensions where ODRL has no equivalent:
- `adalbert:State` -- unified lifecycle states (Pending/Active/Fulfilled/Violated)
- `adalbert:deadline` -- deadline semantics on duties
- `adalbert:DataContract` / `adalbert:Subscription` -- data contract lifecycle
- `adalbert:partOf` -- asset hierarchy (on `odrl:Asset`)
- `adalbert:resolutionPath` -- structured operand resolution (on `odrl:LeftOperand`)
- `adalbert:RuntimeReference` -- runtime value references
- `adalbert:not` -- logical negation of constraints

Adalbert policies are valid ODRL 2.2 policies. Adalbert-aware processors are needed only for the extensions above.

### 2. Bilateral Agreements

ODRL evaluates agreements from assignee perspective only. Adalbert returns duties for **both** assigner and assignee, enabling provider SLAs alongside consumer obligations.

### 3. Deadline Semantics

Supports both:
- `xsd:dateTime`: absolute deadline (2026-12-31T23:59:59Z)
- `xsd:duration`: relative to activation (P30D, PT24H)

### 4. ODRL Policy Types

Policies use standard ODRL types directly:
- `odrl:Set`: no parties
- `odrl:Offer`: assigner required, assignee optional
- `odrl:Agreement`: both required

`adalbert:DataContract` subclasses `odrl:Offer`; `adalbert:Subscription` subclasses `odrl:Agreement`.

### 5. Unified State

`adalbert:State` replaces separate duty and contract state enums. Four states (Pending, Active, Fulfilled, Violated) apply to both `odrl:Duty` instances and contracts. Draft/Published are pre-normative workflow metadata, not modeled in the ontology.

### 6. DCON Alignment

DCON concepts (`DataContract`, `Subscription`) are absorbed into core. Alignment with external DCON uses `skos:closeMatch` mappings in `ontology/adalbert-dcon-alignment.ttl`.

### 7. Structured Operand Resolution

`adalbert:resolutionPath` (on `odrl:LeftOperand`) replaces `contextKey`. Paths are dot-separated from canonical roots (`agent`, `asset`, `context`), validated by SHACL.

---

## Namespaces

```
odrl:          http://www.w3.org/ns/odrl/2/        (primary -- all standard constructs)
adalbert:      https://vocabulary.bigbank/adalbert/  (extensions only)
adalbert-due:  https://vocabulary.bigbank/adalbert/due/
```

`odrl:` is the primary namespace. Adalbert uses ODRL terms wherever possible. The `adalbert:` namespace is reserved for extensions that have no ODRL equivalent (lifecycle state, deadline, data contracts, hierarchy, operand resolution, runtime references, logical negation).

See [../config/namespaces.ttl](../config/namespaces.ttl) for authoritative registry.
