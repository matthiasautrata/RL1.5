# Adalbert Profiles

Domain-specific vocabularies extending Adalbert Core.

## Profile Architecture

```
adalbert-core.ttl
(ODRL profile extension:
 State, deadline, recurrence,
 subject, object,
 partOf, memberOf,
 resolutionPath,
 RuntimeReference, not)
       |
       v
  adalbert-due.ttl
  (all operands, actions,
   concept values)
```

## Profiles

| Profile | Namespace | Description |
|---------|-----------|-------------|
| **Data Use (DUE)** | `adalbert-due:` + `odrl:` | Operands, DUE-specific actions, concept values. Uses ODRL Common Vocabulary actions directly. |

## Design

DUE is the single vocabulary profile for data-use evaluation requests.

DUE uses ODRL Common Vocabulary actions directly where ODRL defines them (`odrl:use`, `odrl:read`, `odrl:display`, `odrl:distribute`, `odrl:delete`, `odrl:modify`, `odrl:aggregate`, `odrl:anonymize`, `odrl:derive`). DUE-specific actions that have no ODRL equivalent use the `adalbert-due:` namespace (`nonDisplay`, `conformTo`, `log`, `notify`, `report`, `deliver`, `calculateIndex`, `algorithmicTrading`, `query`, `export`, `copy`, `link`, `profile`).

## Extension Mechanism

New domain profiles can be added for specialized vocabularies (e.g., healthcare, media licensing). The process:

1. Create `profiles/adalbert-<domain>.ttl`
2. Import `adalbert:` (core)
3. Define domain-specific operands as `odrl:LeftOperand` with `adalbert:resolutionPath`
4. Define actions as `odrl:Action` with `odrl:includedIn` hierarchy
5. Define concept values as `skos:Concept`
6. Register in `ontology/adalbert-prof.ttl`

Profiles extend vocabulary only. They never add norm types, policy types, or evaluation rules.

## resolutionPath Convention

Every operand must have a `resolutionPath` starting with a canonical root:

| Root | Meaning | Examples |
|------|---------|----------|
| `agent` | Requesting agent | `agent.role`, `agent.organization` |
| `asset` | Target asset | `asset.classification`, `asset.market` |
| `context` | Request context | `context.purpose`, `context.environment` |
