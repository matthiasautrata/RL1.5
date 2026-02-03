# Adalbert DCON Profile vs DCON Ontology

**Purpose**: Analyze alignment and differences between Adalbert's DCON conformance declaration and the DCON ontology.

---

## Overview

| Aspect | DCON Ontology | Adalbert DCON Profile |
|--------|---------------|-------------------|
| **Purpose** | Data contract vocabulary | Adalbert conformance for DCON |
| **Scope** | Full contract lifecycle | Policy semantics subset |
| **Promise model** | Simplified lifecycle | Deferred to RL2 |
| **Integration** | Standalone vocabulary | Profile of Adalbert |

---

## Architectural Relationship

```
                    ODRL 2.2 Core
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
       Adalbert          DCON          Other ODRL
    (Semantics)    (Vocabulary)     Profiles
          │              │
          └──────┬───────┘
                 │
                 ▼
          Adalbert DCON Profile
         (Intersection)
```

**Key insight**: Adalbert DCON Profile is not a copy of DCON; it's the intersection of DCON concepts that can be expressed with Adalbert semantics.

---

## Promise Theory Comparison

### DCON Promise Model

DCON defines a simplified Promise lifecycle:

```turtle
dcon:Promise a owl:Class ;
    owl:disjointWith odrl:Rule, odrl:Duty, odrl:Permission, odrl:Prohibition ;
    skos:definition "A voluntary commitment from a promisor to a promisee about an outcome or state."@en .

dcon:PromiseState ::= Pending | Fulfilled | Violated
```

### Adalbert Approach

Adalbert defers full Promise Theory to RL2 but provides:

1. **Duty lifecycle** — Covers the "ought-to-do" semantics
2. **Condition semantics** — Covers "ought-to-be" pre-requisites
3. **Agreement structure** — Covers bilateral commitment formation

### Mapping

| DCON Concept | Adalbert Equivalent | Notes |
|--------------|------------------|-------|
| `dcon:Promise` | Deferred to RL2 | Use `adalbert:Duty` for obligations |
| `dcon:ProviderPromise` | `adalbert:Duty` on grantor | Via agreement structure |
| `dcon:promiseState` | `adalbert:dutyState` | Same lifecycle semantics |
| `dcon:promisor` | `adalbert:grantor` | Agreement-level |
| `dcon:promisee` | `adalbert:grantee` | Agreement-level |
| `dcon:promiseContent` | `adalbert:action` + conditions | Decomposed |

---

## Policy Type Comparison

### DCON Policy Types

```turtle
dcon:DataContract rdfs:subClassOf odrl:Offer ;
    skos:definition "A standing offer specifying data assets, usage permissions, and provider promises."@en .

dcon:DataContractSubscription rdfs:subClassOf odrl:Agreement ;
    skos:definition "An agreement recording a consumer's acceptance of a data contract."@en .
```

### Adalbert Policy Types

```turtle
adalbert:Set a owl:Class ;
    rdfs:subClassOf odrl:Set ;
    skos:definition "A named collection of rules without party binding."@en .

adalbert:Agreement a owl:Class ;
    rdfs:subClassOf odrl:Agreement ;
    skos:definition "A binding agreement between grantor and grantee."@en .
```

### Mapping

| DCON | Adalbert DCON Profile | Notes |
|------|-------------------|-------|
| `dcon:DataContract` | N/A | DCON-specific (Offer) |
| `dcon:DataContractSubscription` | `adalbert:Agreement` | When activated |

**Key difference**: DCON distinguishes Offer (DataContract) from Agreement (Subscription). Adalbert only models Agreement; DCON handles offer semantics.

---

## Property Mappings

### Core Properties

| DCON Property | Adalbert Equivalent | Mapping Type |
|---------------|------------------|--------------|
| `dcon:hasPromise` | N/A | Deferred |
| `odrl:permission` | `adalbert:clause` (Privilege) | Structural |
| `odrl:obligation` | `adalbert:clause` (Duty) | Structural |
| `odrl:target` | `adalbert:object` | Renamed |
| `dcat:distribution` | N/A | Use DCAT directly |

### Promise Properties (Deferred in Adalbert)

| DCON Property | Adalbert Status | Alternative |
|---------------|--------------|-------------|
| `dcon:promisor` | Deferred | Use `adalbert:grantor` on Agreement |
| `dcon:promisee` | Deferred | Use `adalbert:grantee` on Agreement |
| `dcon:promiseContent` | Deferred | Use `adalbert:action` + `adalbert:condition` |
| `dcon:promiseState` | Partial | `adalbert:dutyState` for duties |

### Specialized Promise Types

| DCON Class | Adalbert Alternative |
|------------|-------------------|
| `dcon:ProviderTimelinessPromise` | `adalbert:Duty` with `adalbert:deadline` |
| `dcon:ProviderSchemaPromise` | `adalbert:Duty` with `dcterms:conformsTo` condition |
| `dcon:ProviderChangeNotificationPromise` | `adalbert:Duty` with `adalbert-du:notify` action |

---

## Semantic Differences

### 1. Promise vs Duty Distinction

**DCON**: 
> "Promises are 'ought-to-be' commitments about outcomes/states.
> Duties are 'ought-to-do' obligations to perform actions.
> Promises give rise to Duties when agreements are formed."

**Adalbert**: 
> Duties directly represent obligations. Promises are deferred to RL2.

**Implication**: DCON's richer semantic distinction is collapsed in Adalbert. When RL2 is available, DCON can use full Promise semantics.

### 2. Contract Formation

**DCON**:
```
DataContract (Offer) --[acceptance]--> DataContractSubscription (Agreement)
```

**Adalbert**:
```
Agreement (already accepted)
```

**Implication**: Adalbert policies represent agreed terms. DCON handles the offer→acceptance transition.

### 3. Provider Commitments

**DCON** models provider promises explicitly:

```turtle
ex:contract dcon:hasPromise [
    a dcon:ProviderTimelinessPromise ;
    dcon:promisor ex:dataTeam ;
    dcon:promisee ex:analytics ;
    dcon:promiseContent [ ... ] ;
    dcon:promiseState dcon:Pending
] .
```

**Adalbert** models as duties on the grantor:

```turtle
ex:agreement adalbert:grantor ex:dataTeam ;
    adalbert:grantee ex:analytics ;
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:dataTeam ;  # Grantor has the duty
        adalbert:action adalbert-du:deliverData ;
        adalbert:deadline "06:00:00"^^xsd:time ;
        adalbert:dutyState adalbert:Pending
    ] .
```

---

## Integration Pattern

### Using DCON with Adalbert

1. **DCON handles**: Contract lifecycle, offer management, versioning
2. **Adalbert handles**: Policy semantics, evaluation, compliance checking

```turtle
# DCON DataContractSubscription references Adalbert policy
ex:subscription a dcon:DataContractSubscription ;
    prov:wasDerivedFrom ex:contract ;
    dcon:hasAgreedPolicy ex:rl15Policy .

# Adalbert policy with formal semantics
ex:rl15Policy a adalbert:Agreement ;
    odrl:profile <https://rl2.org/adalbert/> ;
    adalbert:grantor ex:dataTeam ;
    adalbert:grantee ex:analytics ;
    # ... clauses with Adalbert semantics
```

### Namespace Alignment

DCON2 namespaces.ttl defines:

```turtle
@prefix dcon: <https://vocabulary.jpmorgan/dcon/> .
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
```

Adalbert adds:

```turtle
@prefix adalbert: <https://rl2.org/adalbert/> .
@prefix rl15-dcon: <https://rl2.org/adalbert/dcon/> .
```

**Coexistence**: Both namespaces can be used in the same document:

```turtle
ex:subscription a dcon:DataContractSubscription ;
    dcon:hasAgreedPolicy [
        a adalbert:Agreement ;
        # Adalbert semantics apply to policy evaluation
    ] .
```

---

## What Adalbert DCON Profile Provides

### 1. Formal Evaluation Semantics

DCON doesn't define how policies are evaluated. Adalbert provides:

```
Eval : Request × Policy × State → Decision × DutySet
```

### 2. Deterministic Conflict Resolution

DCON inherits ODRL's ambiguity. Adalbert fixes it:

```
Prohibition > Privilege (fixed)
```

### 3. Duty Lifecycle Semantics

DCON has `promiseState` but no transition rules. Adalbert provides:

```
Pending --[condition true]--> Active --[action performed]--> Fulfilled
                                    --[deadline passed]--> Violated
```

### 4. Verification Path

Adalbert policies can be formally verified; DCON policies cannot (without Adalbert).

---

## What DCON Provides Beyond Adalbert

### 1. Contract Lifecycle

```
Draft → Published → Subscribed → Active → Terminated
```

### 2. Versioning (prov:wasRevisionOf)

```turtle
ex:contract-v2 prov:wasRevisionOf ex:contract-v1 .
```

### 3. Rich Promise Taxonomy

```
Promise
├── ProviderPromise
│   ├── ProviderTimelinessPromise
│   ├── ProviderSchemaPromise
│   └── ProviderChangeNotificationPromise
└── (Future: ConsumerPromise)
```

### 4. Catalog Integration (DCAT)

```turtle
ex:contract dcat:distribution ex:apiEndpoint .
```

---

## Migration Path

### Current: Adalbert + DCON

- Use DCON for contract structure
- Use Adalbert for policy semantics
- Both work together via integration pattern

### Future: RL2 + DCON

- RL2 provides full Promise Theory
- DCON aligns Promise taxonomy with RL2
- Full semantic compatibility

```
Adalbert --[upgrade]--> RL2
                       │
                       │ (Promise Theory alignment)
                       │
DCON ────────────────[align]
```

---

## Example: Complete Integration

```turtle
@prefix dcon: <https://vocabulary.jpmorgan/dcon/> .
@prefix adalbert: <https://rl2.org/adalbert/> .
@prefix prov: <http://www.w3.org/ns/prov#> .

# DCON Subscription (contract lifecycle)
ex:subscription a dcon:DataContractSubscription ;
    dcon:subscriber ex:analyticsTeam ;
    prov:wasDerivedFrom ex:customerDataContract ;
    dcon:effectiveDate "2025-01-01"^^xsd:date ;
    dcon:hasAgreedPolicy ex:analyticsPolicy .

# Adalbert Policy (formal semantics)
ex:analyticsPolicy a adalbert:Agreement ;
    odrl:profile <https://rl2.org/adalbert/> ;
    adalbert:grantor ex:dataTeam ;
    adalbert:grantee ex:analyticsTeam ;
    
    adalbert:clause [
        a adalbert:Privilege ;
        adalbert:subject ex:analyticsTeam ;
        adalbert:action adalbert-du:derive ;
        adalbert:object ex:customerData ;
        adalbert:condition [
            adalbert:leftOperand adalbert-du:purpose ;
            adalbert:constraintOperator adalbert:eq ;
            adalbert:rightOperand ex:internalAnalytics
        ]
    ] ;
    
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:dataTeam ;  # Provider duty
        adalbert:action adalbert-du:deliverData ;
        adalbert:object ex:customerData ;
        adalbert:deadline "06:00:00"^^xsd:time ;
        adalbert:dutyState adalbert:Pending
    ] .
```

---

## References

- [DCON Ontology](../../dcon/dcon/ontology/dcon.ttl)
- [DCON2 Namespaces](../../dcon/dcon2/config/namespaces.ttl)
- [Adalbert DCON Profile](../profiles/adalbert-dcon.ttl)
- [RL2 Promise Theory](../../RL2/docs/promise-theory.md)
