# Adalbert Overview

What Adalbert is, who it is for, and how to get started.

---

## Quick Start

Adalbert is a **deterministic ODRL 2.2 profile** for data governance. It fixes ODRL's ambiguities — undefined duty lifecycle, unilateral-only agreements, implicit evaluation order — while remaining a proper profile: every Adalbert policy is a valid ODRL 2.2 policy.

Adalbert is designed for organizations that need:

- **Data contracts** between providers and consumers (SLAs, delivery schedules, usage restrictions)
- **Data use policies** governing access by purpose, role, jurisdiction, and classification
- **Deterministic evaluation** where the same request always produces the same decision
- **Formal verification** of policy correctness (amenable to Dafny, Why3, Coq)

### Minimal Data Contract

```turtle
@prefix odrl:         <http://www.w3.org/ns/odrl/2/> .
@prefix adalbert:     <https://vocabulary.bigbank/adalbert/> .
@prefix adalbert-due: <https://vocabulary.bigbank/adalbert/due/> .
@prefix xsd:          <http://www.w3.org/2001/XMLSchema#> .

ex:contract a adalbert:DataContract ;
    odrl:profile <https://vocabulary.bigbank/adalbert/> ;
    odrl:assigner ex:dataTeam ;
    odrl:target ex:marketPrices ;
    odrl:obligation [
        a odrl:Duty ;
        odrl:assignee ex:dataTeam ;
        odrl:action adalbert-due:deliver ;
        odrl:target ex:marketPrices ;
        adalbert:recurrence "FREQ=DAILY;BYHOUR=6;BYMINUTE=0" ;
        adalbert:deadline "PT30M"^^xsd:duration
    ] ;
    odrl:permission [
        a odrl:Permission ;
        odrl:action odrl:display ;
        odrl:target ex:marketPrices
    ] .
```

### Minimal Data Use Policy

```turtle
ex:policy a odrl:Set ;
    odrl:profile <https://vocabulary.bigbank/adalbert/> ;
    odrl:target ex:employeeData ;
    odrl:permission [
        a odrl:Permission ;
        odrl:assignee ex:hrTeam ;
        odrl:action odrl:read ;
        odrl:target ex:employeeData ;
        odrl:constraint [
            a odrl:Constraint ;
            odrl:leftOperand odrl:purpose ;
            odrl:operator odrl:eq ;
            odrl:rightOperand adalbert-due:analytics
        ]
    ] .
```

---

## Key Concepts

### Permission, Duty, Prohibition

Adalbert uses ODRL's three rule types directly:

| Rule Type | Meaning | Example |
|-----------|---------|---------|
| `odrl:Permission` | Grants access under conditions | "Analytics team may read for analytics purpose" |
| `odrl:Duty` | Requires an action | "Provider must deliver data daily by 06:30" |
| `odrl:Prohibition` | Denies access | "No external distribution" |

Prohibitions always override permissions (fixed conflict resolution: `odrl:prohibit`).

### Constraint

Conditions on rules. An `odrl:Constraint` compares a left operand to a right operand via an operator:

```turtle
odrl:constraint [
    a odrl:Constraint ;
    odrl:leftOperand odrl:purpose ;
    odrl:operator odrl:eq ;
    odrl:rightOperand adalbert-due:analytics
] .
```

Constraints can be combined with `odrl:and`, `odrl:or`, or negated with `adalbert:not` via `odrl:LogicalConstraint`.

### State

Duties and contracts share a four-state lifecycle:

```
         condition true
Pending ──────────────> Active
                         |  |
          action done    |  |  deadline passed
                         v  v
                   Fulfilled  Violated
```

### DataContract and Subscription

An `adalbert:DataContract` (subclass of `odrl:Offer`) is a provider's offer specifying SLAs, permissions, and restrictions. An `adalbert:Subscription` (subclass of `odrl:Agreement`) is an activated contract binding both parties.

### Recurrence

Recurring duties use `adalbert:recurrence` — an RFC 5545 RRULE string. Combined with `adalbert:deadline`, this defines a schedule and per-instance fulfillment window. Each generated instance follows the lifecycle independently.

---

## Architecture

Adalbert has a layered architecture:

```
ODRL 2.2 Core (W3C Standard)
  Permission, Duty, Prohibition, Set, Offer, Agreement,
  Constraint, Action, Asset, Party, LeftOperand, operators
         |
         | proper profile (thin extension)
         v
Adalbert Core (adalbert:)
  State, deadline, recurrence, DataContract, Subscription,
  partOf, memberOf, resolutionPath, RuntimeReference, not
         |
         | domain vocabulary
         v
Adalbert DUE Profile (adalbert-due:)
  50+ operands (purpose, classification, jurisdiction, ...)
  15+ actions (deliver, notify, conformTo, nonDisplay, ...)
  SKOS concept values (analytics, confidential, PII, ...)
```

### Namespaces

| Prefix | Namespace | Role |
|--------|-----------|------|
| `odrl:` | `http://www.w3.org/ns/odrl/2/` | Primary — all standard constructs |
| `adalbert:` | `https://vocabulary.bigbank/adalbert/` | Extensions only |
| `adalbert-due:` | `https://vocabulary.bigbank/adalbert/due/` | Data use vocabulary |

See `config/namespaces.ttl` for the authoritative registry.

---

## How It Differs from ODRL 2.2

ODRL 2.2 is a flexible framework. Adalbert makes it deterministic and governance-ready:

| Aspect | ODRL 2.2 | Adalbert |
|--------|----------|----------|
| Duty lifecycle | Undefined | Pending -> Active -> Fulfilled/Violated |
| Agreement evaluation | Assignee duties only | Both assigner and assignee duties (bilateral) |
| Conflict resolution | Configurable | Fixed: Prohibition > Permission |
| Evaluation order | Undefined | Deterministic left-to-right |
| Operand resolution | Implicit | Explicit `resolutionPath` from canonical roots |
| Recurring duties | Not supported | `recurrence` (RFC 5545 RRULE) + `deadline` |
| Contract types | Generic Offer/Agreement | `DataContract` (Offer) / `Subscription` (Agreement) |
| Logical negation | Not supported | `adalbert:not` on LogicalConstraint |

Every Adalbert policy remains a valid ODRL 2.2 policy. Standard ODRL processors can parse them; Adalbert-aware processors additionally enforce lifecycle, bilateral duties, and deterministic evaluation.

---

## DCON Migration

As of v0.7, Adalbert **supersedes** DCON. DCON's promise hierarchy dissolves into standard `odrl:Duty` patterns with DUE actions. If migrating from DCON:

- `dcon:DataContract` -> `adalbert:DataContract`
- `dcon:DataContractSubscription` -> `adalbert:Subscription`
- `dcon:Promise` hierarchy -> `odrl:Duty` with DUE actions (`deliver`, `notify`, `conformTo`, `report`)
- `dcon:promisedDeliveryTime` -> `adalbert:recurrence` + `adalbert:deadline`

See [adalbert-term-mapping.md](adalbert-term-mapping.md) for complete DCON -> Adalbert property mapping and [comparisons/comparison-dcon.md](comparisons/comparison-dcon.md) for the supersession analysis.

---

## Document Map

Which document to read next depends on your role:

| Role | Start Here | Then Read |
|------|-----------|-----------|
| **Contract writer** (data platform team) | [contracts-guide.md](contracts-guide.md) | [examples/data-contract.ttl](../examples/data-contract.ttl) |
| **Policy writer** (data steward) | [policy-writers-guide.md](policy-writers-guide.md) | [examples/data-use-policy.ttl](../examples/data-use-policy.ttl) |
| **Implementer** (runtime developer) | [Adalbert_Semantics.md](Adalbert_Semantics.md) | [adalbert-specification.md](adalbert-specification.md) |
| **Migrating from DCON** | [adalbert-term-mapping.md](adalbert-term-mapping.md) | [comparisons/comparison-dcon.md](comparisons/comparison-dcon.md) |
| **Evaluating Adalbert** | This document | [comparisons/comparison-odrl22.md](comparisons/comparison-odrl22.md) |

### Full Document Inventory

| Document | Description |
|----------|-------------|
| [adalbert-overview.md](adalbert-overview.md) | What is Adalbert? (this document) |
| [adalbert-specification.md](adalbert-specification.md) | Technical vocabulary reference |
| [adalbert-term-mapping.md](adalbert-term-mapping.md) | Business term -> property mapping + DCON migration |
| [contracts-guide.md](contracts-guide.md) | Data contract authoring guide |
| [policy-writers-guide.md](policy-writers-guide.md) | Data use policy authoring guide |
| [Adalbert_Semantics.md](Adalbert_Semantics.md) | Formal operational semantics (normative) |

---

**Version**: 0.7 | **Date**: 2026-02-04
