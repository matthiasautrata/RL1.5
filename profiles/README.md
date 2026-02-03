# RL1.5 Profiles

Domain-specific vocabularies extending RL1.5 Core.

## Profile Hierarchy

```
rl1.5 (core)
├── rl1.5/governance (shared operands)
│   ├── rl1.5/market-data
│   └── rl1.5/data-use
└── rl1.5/contracts (extension)
```

## Profiles

| Profile | Namespace | Description |
|---------|-----------|-------------|
| **Governance Core** | `rl15-gov:` | Shared operands: purpose, classification, jurisdiction |
| **Market Data** | `rl15-md:` | Timeliness, display/non-display, recipient types |
| **Data Use** | `rl15-du:` | Role-based access, environment, consent |
| **Contracts** | `rl15-dc:` | DataContract, Subscription, lifecycle states |

## Creating a Profile

1. Declare as `owl:Ontology` and `odrl:Profile`
2. Import governance core (or core directly)
3. Define operands as `rl15:LeftOperand` with `rl15:contextKey`
4. Define actions as `rl15:Action` with `rl15:includedIn` hierarchy
5. Define concept values as `skos:Concept`

Example:

```turtle
@prefix rl15:    <https://vocabulary.bigbank/rl1.5/> .
@prefix ex:      <https://vocabulary.bigbank/rl1.5/example/> .

<https://vocabulary.bigbank/rl1.5/example/> a owl:Ontology, odrl:Profile ;
    owl:imports <https://vocabulary.bigbank/rl1.5/governance/> .

ex:myOperand a rl15:LeftOperand ;
    rdfs:label "my operand"@en ;
    rl15:contextKey "myOperand" .

ex:myAction a rl15:Action ;
    rdfs:label "my action"@en ;
    rl15:includedIn rl15-gov:use .
```

## DCON Alignment

The Contracts extension (`rl15-dc:`) absorbs key DCON concepts:

| DCON | RL1.5 Contracts | Relationship |
|------|-----------------|--------------|
| `dcon:DataContract` | `rl15-dc:DataContract` | `skos:closeMatch` |
| `dcon:DataContractSubscription` | `rl15-dc:Subscription` | `skos:closeMatch` |
| `dcon:Draft/Published/Active` | `rl15-dc:Draft/Published/Active` | `skos:closeMatch` |

Note: State mappings use `skos:closeMatch`, not `owl:sameAs`, because the state machines differ (RL1.5 duties have 4 states vs DCON's 3).
