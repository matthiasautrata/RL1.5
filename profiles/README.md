# Adalbert Profiles

Domain-specific vocabularies extending Adalbert Core.

## Profile Architecture

```
adalbert-core.ttl
(ODRL profile extension:
 State, deadline, recurrence,
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

Contract lifecycle classes (`DataContract`, `Subscription`) and their properties (`subscribesTo`, `effectiveDate`, `expirationDate`) live in core, not in a profile. DCON is superseded as of v0.7 — the alignment file is deprecated (historical reference only).

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

## DCON Supersession

As of v0.7, Adalbert supersedes DCON. DCON's promise hierarchy dissolves into `odrl:Duty` patterns with DUE actions. See `docs/comparisons/comparison-dcon.md` for the supersession analysis.

| DCON Concept | Adalbert v0.7 | Status |
|--------------|---------------|--------|
| `dcon:DataContract` | `adalbert:DataContract` | Absorbed into core |
| `dcon:DataContractSubscription` | `adalbert:Subscription` | Absorbed into core |
| `dcon:Promise` hierarchy | `odrl:Duty` + DUE actions | Dissolved |
| Promise scheduling | `adalbert:recurrence` + `adalbert:deadline` | New in v0.7 |

See [comparison-dcon.md](../docs/comparisons/comparison-dcon.md) for the full supersession analysis.
