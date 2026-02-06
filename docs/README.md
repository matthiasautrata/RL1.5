# Adalbert Documentation

## Guides

| Document | Description |
|----------|-------------|
| [adalbert-overview.md](adalbert-overview.md) | What is Adalbert? Architecture, key concepts, document map |
| [contracts-guide.md](contracts-guide.md) | Data contract authoring guide (DataContract, Subscription, recurrence, patterns) |
| [policy-writers-guide.md](policy-writers-guide.md) | Data use policy authoring guide (permissions, prohibitions, constraints, DUE vocabulary) |

## Reference

| Document | Description |
|----------|-------------|
| [Adalbert_Semantics.md](Adalbert_Semantics.md) | Formal semantics (normative) |
| [adalbert-specification.md](adalbert-specification.md) | Technical vocabulary reference (classes, properties, SHACL, DUE summary) |
| [adalbert-term-mapping.md](adalbert-term-mapping.md) | Business term -> property mapping + DCON migration |

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
| [comparison-dcon.md](comparisons/comparison-dcon.md) | DCON supersession analysis (+ completeness verification) |
| [comparison-rl2.md](comparisons/comparison-rl2.md) | vs RL2 (functional gap, complexity, translation path) |
| [namespace-alignment.md](comparisons/namespace-alignment.md) | Namespace coordination |

## Examples

| File | Description |
|------|-------------|
| [data-contract.ttl](../examples/data-contract.ttl) | Complete contract example (bilateral duties, recurrence, subscription) |
| [data-use-policy.ttl](../examples/data-use-policy.ttl) | Access control example (role-based, purpose constraints) |
| [baseline.ttl](../examples/baseline.ttl) | Comprehensive test data (5 orgs, 8 assets, 8 contracts, 2 subscriptions) |

---

## Key Design Decisions (v0.7)

### 1. Proper ODRL 2.2 Profile

Adalbert is a proper **ODRL 2.2 profile**. It uses ODRL terms for all standard constructs (`odrl:Permission`, `odrl:Duty`, `odrl:Prohibition`, `odrl:target`, `odrl:Set`, `odrl:Offer`, `odrl:Agreement`, `odrl:LeftOperand`, etc.) and only introduces extensions where ODRL has no equivalent:
- `adalbert:State` -- unified lifecycle states (Pending/Active/Fulfilled/Violated)
- `adalbert:deadline` -- deadline semantics on duties
- `adalbert:recurrence` -- RFC 5545 RRULE for recurring duties
- `adalbert:subject` -- duty bearer party role (on `odrl:Duty`, `rdfs:subPropertyOf odrl:assignee`)
- `adalbert:object` -- affected party role (on `odrl:Duty`, `rdfs:subPropertyOf odrl:function`)
- `adalbert:DataContract` / `adalbert:Subscription` -- data contract lifecycle
- `adalbert:partOf` -- asset hierarchy (on `odrl:Asset`, `rdfs:subPropertyOf odrl:partOf`)
- `adalbert:memberOf` -- party hierarchy (on `odrl:Party`, `rdfs:subPropertyOf odrl:partOf`)
- `adalbert:resolutionPath` -- structured operand resolution (on `odrl:LeftOperand`)
- `adalbert:RuntimeReference` -- runtime value references
- `adalbert:not` -- logical negation of constraints

Adalbert policies are valid ODRL 2.2 policies. Adalbert-aware processors are needed only for the extensions above.

### 2. Bilateral Agreements

ODRL evaluates agreements from assignee perspective only. Adalbert returns duties for **both** assigner and assignee, enabling provider SLAs alongside consumer obligations.

### 3. Deadline Semantics

Supports two forms:
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

### 6. DCON Supersession

As of v0.7, Adalbert supersedes DCON. DCON's promise hierarchy dissolves into `odrl:Duty` patterns with DUE actions. See `comparisons/comparison-dcon.md` for the supersession analysis.

### 7. Structured Operand Resolution

`adalbert:resolutionPath` (on `odrl:LeftOperand`) replaces `contextKey`. Paths are dot-separated from canonical roots (`agent`, `asset`, `context`), validated by SHACL.

### 8. Recurrence (v0.7)

`adalbert:recurrence` is an RFC 5545 RRULE string on `odrl:Duty`. It defines when duty instances are generated; `deadline` defines the per-instance fulfillment window. Single property, any iCal library can parse it, avoids a class hierarchy. Actions stay in DUE (`deliver`, `notify`, `conformTo`), not core.

---

## Namespaces

```
odrl:          http://www.w3.org/ns/odrl/2/        (primary -- all standard constructs)
adalbert:      https://vocabulary.bigbank/adalbert/  (extensions only)
adalbert-due:  https://vocabulary.bigbank/adalbert/due/
```

`odrl:` is the primary namespace. Adalbert uses ODRL terms wherever possible. The `adalbert:` namespace is reserved for extensions that have no ODRL equivalent (lifecycle state, deadline, recurrence, data contracts, hierarchy, operand resolution, runtime references, logical negation).

See [../config/namespaces.ttl](../config/namespaces.ttl) for authoritative registry.
