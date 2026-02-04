# Adalbert Profiles

Domain-specific vocabularies extending Adalbert Core.

## Profile Architecture

```
adalbert-core.ttl
(ODRL profile extension:
 State, deadline,
 DataContract, Subscription,
 partOf, memberOf,
 resolutionPath,
 RuntimeReference, not)
       │
       ▼
  adalbert-due.ttl
  (all operands, actions,
   concept values)
```

## Profiles

| Profile | Namespace | Description |
|---------|-----------|-------------|
| **Data Use (DUE)** | `adalbert-due:` + `odrl:` | Operands, DUE-specific actions, concept values. Uses ODRL Common Vocabulary actions directly. |

## Design

DUE is the single vocabulary profile — it defines what you can/can't do with data. Used in both `odrl:Set` policies (org-wide rules) and as rule content within contracts.

DUE uses ODRL Common Vocabulary actions directly where ODRL defines them (`odrl:use`, `odrl:read`, `odrl:display`, `odrl:distribute`, `odrl:delete`, `odrl:modify`, `odrl:aggregate`, `odrl:anonymize`, `odrl:derive`). DUE-specific actions that have no ODRL equivalent use the `adalbert-due:` namespace (`nonDisplay`, `conformTo`, `log`, `notify`, `report`, `deliver`, `calculateIndex`, `algorithmicTrading`, `query`, `export`, `copy`, `link`, `profile`).

Contract lifecycle classes (`DataContract`, `Subscription`) and their properties (`subscribesTo`, `effectiveDate`, `expirationDate`) live in core, not in a profile. DCON alignment mappings live in `ontology/adalbert-dcon-alignment.ttl`.

External vendor contracts (MDS, Bloomberg, etc.) are captured as `adalbert:DataContract` offers using DUE vocabulary. Internal teams subscribe to those contracts and are additionally governed by org-wide DUE Set policies.

## Extension Mechanism

New domain profiles can be added for specialized vocabularies (e.g., healthcare, media licensing). The process:

1. Create `profiles/adalbert-<domain>.ttl`
2. Import `adalbert:` (core)
3. Define domain-specific operands as `odrl:LeftOperand` with `adalbert:resolutionPath`
4. Define actions as `odrl:Action` with `odrl:includedIn` hierarchy
5. Define concept values as `skos:Concept`
6. Register in `ontology/adalbert-prof.ttl`

Profiles extend vocabulary only — they never add norm types, policy types, or evaluation rules.

## Creating a Profile

1. Declare as `owl:Ontology` and `odrl:Profile`
2. Import core (`adalbert:`)
3. Define operands as `odrl:LeftOperand` with `adalbert:resolutionPath`
4. Define actions as `odrl:Action` with `odrl:includedIn` hierarchy
5. Define concept values as `skos:Concept`

### resolutionPath Convention

Every operand must have a `resolutionPath` starting with a canonical root:

| Root | Meaning | Examples |
|------|---------|----------|
| `agent` | Requesting agent | `agent.role`, `agent.organization` |
| `asset` | Target asset | `asset.classification`, `asset.market` |
| `context` | Request context | `context.purpose`, `context.environment` |

## DCON Alignment

DCON alignment is not a profile but a mapping file (`ontology/adalbert-dcon-alignment.ttl`). It provides `skos:closeMatch` mappings between Adalbert core concepts and DCON:

| Adalbert | DCON | Relationship |
|----------|------|--------------|
| `adalbert:DataContract` | `dcon:DataContract` | `skos:closeMatch` |
| `adalbert:Subscription` | `dcon:DataContractSubscription` | `skos:closeMatch` |
| `odrl:Duty` | `dcon:Duty` | `skos:closeMatch` |
| `adalbert:State` instances | DCON duty states | `skos:closeMatch` |

Note: State mappings use `skos:closeMatch`, not `owl:sameAs`, because the state machines differ (Adalbert has 4 unified states vs DCON's separate contract/promise states).
