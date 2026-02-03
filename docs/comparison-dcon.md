# RL1.5 DCON Profile vs DCON Ontology

**Purpose**: Analyze alignment and differences between RL1.5's DCON conformance declaration and the DCON ontology.

---

## Overview

| Aspect | DCON Ontology | RL1.5 DCON Profile |
|--------|---------------|-------------------|
| **Purpose** | Data contract vocabulary | RL1.5 conformance for DCON |
| **Scope** | Full contract lifecycle | Policy semantics subset |
| **Promise model** | Simplified lifecycle | Deferred to RL2 |
| **Integration** | Standalone vocabulary | Profile of RL1.5 |

---

## Architectural Relationship

```
                    ODRL 2.2 Core
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
       RL1.5          DCON          Other ODRL
    (Semantics)    (Vocabulary)     Profiles
          │              │
          └──────┬───────┘
                 │
                 ▼
          RL1.5 DCON Profile
         (Intersection)
```

**Key insight**: RL1.5 DCON Profile is not a copy of DCON; it's the intersection of DCON concepts that can be expressed with RL1.5 semantics.

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

### RL1.5 Approach

RL1.5 defers full Promise Theory to RL2 but provides:

1. **Duty lifecycle** — Covers the "ought-to-do" semantics
2. **Condition semantics** — Covers "ought-to-be" pre-requisites
3. **Agreement structure** — Covers bilateral commitment formation

### Mapping

| DCON Concept | RL1.5 Equivalent | Notes |
|--------------|------------------|-------|
| `dcon:Promise` | Deferred to RL2 | Use `rl15:Duty` for obligations |
| `dcon:ProviderPromise` | `rl15:Duty` on grantor | Via agreement structure |
| `dcon:promiseState` | `rl15:dutyState` | Same lifecycle semantics |
| `dcon:promisor` | `rl15:grantor` | Agreement-level |
| `dcon:promisee` | `rl15:grantee` | Agreement-level |
| `dcon:promiseContent` | `rl15:action` + conditions | Decomposed |

---

## Policy Type Comparison

### DCON Policy Types

```turtle
dcon:DataContract rdfs:subClassOf odrl:Offer ;
    skos:definition "A standing offer specifying data assets, usage permissions, and provider promises."@en .

dcon:DataContractSubscription rdfs:subClassOf odrl:Agreement ;
    skos:definition "An agreement recording a consumer's acceptance of a data contract."@en .
```

### RL1.5 Policy Types

```turtle
rl15:Set a owl:Class ;
    rdfs:subClassOf odrl:Set ;
    skos:definition "A named collection of rules without party binding."@en .

rl15:Agreement a owl:Class ;
    rdfs:subClassOf odrl:Agreement ;
    skos:definition "A binding agreement between grantor and grantee."@en .
```

### Mapping

| DCON | RL1.5 DCON Profile | Notes |
|------|-------------------|-------|
| `dcon:DataContract` | N/A | DCON-specific (Offer) |
| `dcon:DataContractSubscription` | `rl15:Agreement` | When activated |

**Key difference**: DCON distinguishes Offer (DataContract) from Agreement (Subscription). RL1.5 only models Agreement; DCON handles offer semantics.

---

## Property Mappings

### Core Properties

| DCON Property | RL1.5 Equivalent | Mapping Type |
|---------------|------------------|--------------|
| `dcon:hasPromise` | N/A | Deferred |
| `odrl:permission` | `rl15:clause` (Privilege) | Structural |
| `odrl:obligation` | `rl15:clause` (Duty) | Structural |
| `odrl:target` | `rl15:object` | Renamed |
| `dcat:distribution` | N/A | Use DCAT directly |

### Promise Properties (Deferred in RL1.5)

| DCON Property | RL1.5 Status | Alternative |
|---------------|--------------|-------------|
| `dcon:promisor` | Deferred | Use `rl15:grantor` on Agreement |
| `dcon:promisee` | Deferred | Use `rl15:grantee` on Agreement |
| `dcon:promiseContent` | Deferred | Use `rl15:action` + `rl15:condition` |
| `dcon:promiseState` | Partial | `rl15:dutyState` for duties |

### Specialized Promise Types

| DCON Class | RL1.5 Alternative |
|------------|-------------------|
| `dcon:ProviderTimelinessPromise` | `rl15:Duty` with `rl15:deadline` |
| `dcon:ProviderSchemaPromise` | `rl15:Duty` with `dcterms:conformsTo` condition |
| `dcon:ProviderChangeNotificationPromise` | `rl15:Duty` with `rl15-du:notify` action |

---

## Semantic Differences

### 1. Promise vs Duty Distinction

**DCON**: 
> "Promises are 'ought-to-be' commitments about outcomes/states.
> Duties are 'ought-to-do' obligations to perform actions.
> Promises give rise to Duties when agreements are formed."

**RL1.5**: 
> Duties directly represent obligations. Promises are deferred to RL2.

**Implication**: DCON's richer semantic distinction is collapsed in RL1.5. When RL2 is available, DCON can use full Promise semantics.

### 2. Contract Formation

**DCON**:
```
DataContract (Offer) --[acceptance]--> DataContractSubscription (Agreement)
```

**RL1.5**:
```
Agreement (already accepted)
```

**Implication**: RL1.5 policies represent agreed terms. DCON handles the offer→acceptance transition.

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

**RL1.5** models as duties on the grantor:

```turtle
ex:agreement rl15:grantor ex:dataTeam ;
    rl15:grantee ex:analytics ;
    rl15:clause [
        a rl15:Duty ;
        rl15:subject ex:dataTeam ;  # Grantor has the duty
        rl15:action rl15-du:deliverData ;
        rl15:deadline "06:00:00"^^xsd:time ;
        rl15:dutyState rl15:Pending
    ] .
```

---

## Integration Pattern

### Using DCON with RL1.5

1. **DCON handles**: Contract lifecycle, offer management, versioning
2. **RL1.5 handles**: Policy semantics, evaluation, compliance checking

```turtle
# DCON DataContractSubscription references RL1.5 policy
ex:subscription a dcon:DataContractSubscription ;
    prov:wasDerivedFrom ex:contract ;
    dcon:hasAgreedPolicy ex:rl15Policy .

# RL1.5 policy with formal semantics
ex:rl15Policy a rl15:Agreement ;
    odrl:profile <https://rl2.org/rl1.5/> ;
    rl15:grantor ex:dataTeam ;
    rl15:grantee ex:analytics ;
    # ... clauses with RL1.5 semantics
```

### Namespace Alignment

DCON2 namespaces.ttl defines:

```turtle
@prefix dcon: <https://vocabulary.jpmorgan/dcon/> .
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
```

RL1.5 adds:

```turtle
@prefix rl15: <https://rl2.org/rl1.5/> .
@prefix rl15-dcon: <https://rl2.org/rl1.5/dcon/> .
```

**Coexistence**: Both namespaces can be used in the same document:

```turtle
ex:subscription a dcon:DataContractSubscription ;
    dcon:hasAgreedPolicy [
        a rl15:Agreement ;
        # RL1.5 semantics apply to policy evaluation
    ] .
```

---

## What RL1.5 DCON Profile Provides

### 1. Formal Evaluation Semantics

DCON doesn't define how policies are evaluated. RL1.5 provides:

```
Eval : Request × Policy × State → Decision × DutySet
```

### 2. Deterministic Conflict Resolution

DCON inherits ODRL's ambiguity. RL1.5 fixes it:

```
Prohibition > Privilege (fixed)
```

### 3. Duty Lifecycle Semantics

DCON has `promiseState` but no transition rules. RL1.5 provides:

```
Pending --[condition true]--> Active --[action performed]--> Fulfilled
                                    --[deadline passed]--> Violated
```

### 4. Verification Path

RL1.5 policies can be formally verified; DCON policies cannot (without RL1.5).

---

## What DCON Provides Beyond RL1.5

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

### Current: RL1.5 + DCON

- Use DCON for contract structure
- Use RL1.5 for policy semantics
- Both work together via integration pattern

### Future: RL2 + DCON

- RL2 provides full Promise Theory
- DCON aligns Promise taxonomy with RL2
- Full semantic compatibility

```
RL1.5 --[upgrade]--> RL2
                       │
                       │ (Promise Theory alignment)
                       │
DCON ────────────────[align]
```

---

## Example: Complete Integration

```turtle
@prefix dcon: <https://vocabulary.jpmorgan/dcon/> .
@prefix rl15: <https://rl2.org/rl1.5/> .
@prefix prov: <http://www.w3.org/ns/prov#> .

# DCON Subscription (contract lifecycle)
ex:subscription a dcon:DataContractSubscription ;
    dcon:subscriber ex:analyticsTeam ;
    prov:wasDerivedFrom ex:customerDataContract ;
    dcon:effectiveDate "2025-01-01"^^xsd:date ;
    dcon:hasAgreedPolicy ex:analyticsPolicy .

# RL1.5 Policy (formal semantics)
ex:analyticsPolicy a rl15:Agreement ;
    odrl:profile <https://rl2.org/rl1.5/> ;
    rl15:grantor ex:dataTeam ;
    rl15:grantee ex:analyticsTeam ;
    
    rl15:clause [
        a rl15:Privilege ;
        rl15:subject ex:analyticsTeam ;
        rl15:action rl15-du:derive ;
        rl15:object ex:customerData ;
        rl15:condition [
            rl15:leftOperand rl15-du:purpose ;
            rl15:constraintOperator rl15:eq ;
            rl15:rightOperand ex:internalAnalytics
        ]
    ] ;
    
    rl15:clause [
        a rl15:Duty ;
        rl15:subject ex:dataTeam ;  # Provider duty
        rl15:action rl15-du:deliverData ;
        rl15:object ex:customerData ;
        rl15:deadline "06:00:00"^^xsd:time ;
        rl15:dutyState rl15:Pending
    ] .
```

---

## References

- [DCON Ontology](../../dcon/dcon/ontology/dcon.ttl)
- [DCON2 Namespaces](../../dcon/dcon2/config/namespaces.ttl)
- [RL1.5 DCON Profile](../profiles/rl1_5-dcon.ttl)
- [RL2 Promise Theory](../../RL2/docs/promise-theory.md)
