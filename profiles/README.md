# Adalbert Profiles

Domain-specific vocabularies extending Adalbert Core.

## Profile Hierarchy

```
adalbert (core)
├── adalbert/governance (shared operands)
│   ├── adalbert/market-data
│   └── adalbert/data-use
└── adalbert/contracts (extension)
```

## Profiles

| Profile | Namespace | Description |
|---------|-----------|-------------|
| **Governance Core** | `adalbert-gov:` | Shared operands: purpose, classification, jurisdiction |
| **Market Data** | `adalbert-md:` | Timeliness, display/non-display, recipient types |
| **Data Use** | `adalbert-du:` | Role-based access, environment, consent |
| **Contracts** | `adalbert-dc:` | DataContract, Subscription, lifecycle states |

## Creating a Profile

1. Declare as `owl:Ontology` and `odrl:Profile`
2. Import governance core (or core directly)
3. Define operands as `adalbert:LeftOperand` with `adalbert:contextKey`
4. Define actions as `adalbert:Action` with `adalbert:includedIn` hierarchy
5. Define concept values as `skos:Concept`

Example:

```turtle
@prefix adalbert:    <https://vocabulary.bigbank/adalbert/> .
@prefix ex:      <https://vocabulary.bigbank/adalbert/example/> .

<https://vocabulary.bigbank/adalbert/example/> a owl:Ontology, odrl:Profile ;
    owl:imports <https://vocabulary.bigbank/adalbert/governance/> .

ex:myOperand a adalbert:LeftOperand ;
    rdfs:label "my operand"@en ;
    adalbert:contextKey "myOperand" .

ex:myAction a adalbert:Action ;
    rdfs:label "my action"@en ;
    adalbert:includedIn adalbert-gov:use .
```

## DCON Alignment

The Contracts extension (`adalbert-dc:`) absorbs key DCON concepts:

| DCON | Adalbert Contracts | Relationship |
|------|-----------------|--------------|
| `dcon:DataContract` | `adalbert-dc:DataContract` | `skos:closeMatch` |
| `dcon:DataContractSubscription` | `adalbert-dc:Subscription` | `skos:closeMatch` |
| `dcon:Draft/Published/Active` | `adalbert-dc:Draft/Published/Active` | `skos:closeMatch` |

Note: State mappings use `skos:closeMatch`, not `owl:sameAs`, because the state machines differ (Adalbert duties have 4 states vs DCON's 3).
