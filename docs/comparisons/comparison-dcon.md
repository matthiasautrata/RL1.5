# Adalbert and DCON Integration Guide

**Purpose**: Complete mapping between DCON ontology and Adalbert, with migration examples.

---

## Executive Summary

| DCON Concept | Adalbert Equivalent | Relationship |
|--------------|---------------------|--------------|
| `dcon:DataContract` | `adalbert:DataContract` | `skos:closeMatch` |
| `dcon:DataContractSubscription` | `adalbert:Subscription` | `skos:closeMatch` |
| `dcon:Promise` | `odrl:Duty` | Partial (see section 3) |
| `dcon:Permission` | `odrl:Permission` | `skos:closeMatch` |
| Promise states | `adalbert:State` | Different state machines |

**Key difference**: DCON has 3 duty states (Pending to Fulfilled/Violated), Adalbert has 4 unified states (Pending to Active to Fulfilled/Violated) shared by duties and contracts.

**Architecture**: DataContract and Subscription live in `adalbert-core.ttl`. DCON alignment mappings live in `ontology/adalbert-dcon-alignment.ttl`.

---

## 1. Policy Type Mapping

### 1.1 DataContract to adalbert:DataContract

Both are Offers from a data provider.

**DCON**:
```turtle
@prefix dcon: <https://vocabulary.bigbank/dcon/> .

ex:marketDataContract a dcon:DataContract ;
    dcon:provider ex:dataTeam ;
    dcon:hasPromise ex:timelinessPromise ;
    odrl:permission ex:displayPermission ;
    odrl:target ex:marketDataFeed ;
    dcon:contractState dcon:Published .
```

**Adalbert**:
```turtle
@prefix odrl:         <http://www.w3.org/ns/odrl/2/> .
@prefix adalbert:     <https://vocabulary.bigbank/adalbert/> .
@prefix adalbert-due: <https://vocabulary.bigbank/adalbert/due/> .

ex:marketDataContract a adalbert:DataContract ;
    odrl:assigner ex:dataTeam ;
    odrl:target ex:marketDataFeed ;
    adalbert:state adalbert:Active ;

    odrl:permission [
        a odrl:Permission ;
        odrl:assignee adalbert:currentAgent ;
        odrl:action adalbert-due:display ;
        odrl:target ex:marketDataFeed
    ] ;

    odrl:obligation [
        a odrl:Duty ;
        odrl:assignee ex:dataTeam ;
        odrl:action adalbert-due:notify ;
        odrl:target ex:schemaChanges ;
        adalbert:deadline "P7D"^^xsd:duration
    ] .
```

### 1.2 DataContractSubscription to adalbert:Subscription

Both are Agreements binding provider and consumer.

**DCON**:
```turtle
ex:analyticsSubscription a dcon:DataContractSubscription ;
    dcon:subscribesTo ex:marketDataContract ;
    dcon:subscriber ex:analyticsTeam ;
    dcon:subscriptionState dcon:Active ;
    dcon:effectiveDate "2026-01-15T00:00:00Z"^^xsd:dateTime .
```

**Adalbert**:
```turtle
ex:analyticsSubscription a adalbert:Subscription ;
    adalbert:subscribesTo ex:marketDataContract ;
    odrl:assigner ex:dataTeam ;
    odrl:assignee ex:analyticsTeam ;
    adalbert:effectiveDate "2026-01-15T00:00:00Z"^^xsd:dateTime ;

    odrl:permission [
        a odrl:Permission ;
        odrl:assignee ex:analyticsTeam ;
        odrl:action adalbert-due:display ;
        odrl:target ex:marketDataFeed
    ] ;

    odrl:obligation [
        a odrl:Duty ;
        odrl:assignee ex:dataTeam ;
        odrl:action adalbert-due:notify ;
        odrl:target ex:schemaChanges ;
        adalbert:deadline "P7D"^^xsd:duration ;
        adalbert:state adalbert:Pending
    ] ;

    odrl:obligation [
        a odrl:Duty ;
        odrl:assignee ex:analyticsTeam ;
        odrl:action adalbert-due:report ;
        odrl:target ex:usageStatistics ;
        adalbert:deadline "P30D"^^xsd:duration ;
        adalbert:state adalbert:Pending
    ] .
```

---

## 2. State Mapping

Adalbert uses a unified `adalbert:State` for both duties and contracts. DCON has separate contract states and promise states.

### 2.1 Unified State (Adalbert) vs Contract State (DCON)

| Adalbert State | Duty meaning | Contract meaning | DCON equivalent |
|----------------|-------------|------------------|-----------------|
| Pending | Condition not yet met | Not yet in force | Close to `dcon:Pending` |
| Active | Condition met, action required | In force | Close to `dcon:Active` |
| Fulfilled | Action performed | Obligations complete | `dcon:Fulfilled` |
| Violated | Deadline passed | Breached | `dcon:Violated` |

DCON's `Draft` and `Published` are pre-normative workflow metadata -- not modeled in Adalbert's ontology.

### 2.2 Promise State Machine Difference

**DCON Promise States** (3 states):
```
Pending ----------------> Fulfilled
    |
    +-------------------> Violated
```

**Adalbert State** (4 states):
```
Pending --[condition]--> Active --[action]--> Fulfilled
                           |
                           +--[deadline]--> Violated
```

**Mapping**:
- DCON `Pending` maps to Adalbert `Pending` OR `Active` (condition may not be explicit in DCON)
- DCON `Fulfilled` = Adalbert `Fulfilled`
- DCON `Violated` = Adalbert `Violated`

---

## 3. Promise to Duty Translation

DCON distinguishes Promises (ought-to-be commitments) from Duties (ought-to-do obligations). Adalbert collapses this distinction: **all obligations are `odrl:Duty`**.

### 3.1 Provider Promise Types

**DCON ProviderTimelinessPromise** to `odrl:Duty`:
```turtle
[   a odrl:Duty ;
    odrl:assignee ex:dataTeam ;
    odrl:action adalbert-due:deliver ;
    odrl:target ex:dailyData ;
    odrl:constraint [
        a odrl:Constraint ;
        odrl:leftOperand adalbert:currentDateTime ;
        odrl:operator odrl:gteq ;
        odrl:rightOperand "06:00:00"^^xsd:time
    ] ;
    adalbert:deadline "07:00:00"^^xsd:time ;
    adalbert:state adalbert:Pending
] .
```

**DCON ProviderSchemaPromise** to `odrl:Duty`:
```turtle
[   a odrl:Duty ;
    odrl:assignee ex:dataTeam ;
    odrl:action adalbert-due:conformTo ;
    odrl:target ex:marketDataSchema ;
    adalbert:state adalbert:Active
] .
```

**DCON ProviderChangeNotificationPromise** to `odrl:Duty`:
```turtle
[   a odrl:Duty ;
    odrl:assignee ex:dataTeam ;
    odrl:action adalbert-due:notify ;
    odrl:target ex:schemaChanges ;
    adalbert:deadline "P7D"^^xsd:duration ;
    adalbert:state adalbert:Pending
] .
```

### 3.2 Consumer Promise Types

**DCON ConsumerUsageReportingPromise** to `odrl:Duty`:
```turtle
[   a odrl:Duty ;
    odrl:assignee ex:analyticsTeam ;
    odrl:action adalbert-due:report ;
    odrl:target ex:usageStatistics ;
    adalbert:deadline "P30D"^^xsd:duration ;
    adalbert:state adalbert:Pending
] .
```

---

## 4. Permission Mapping

Straightforward: `dcon:Permission` / `odrl:Permission` maps directly to `odrl:Permission`.

**DCON**:
```turtle
ex:displayPermission a odrl:Permission ;
    odrl:action odrl:display ;
    odrl:target ex:marketDataFeed ;
    odrl:constraint [
        odrl:leftOperand dcon:recipientType ;
        odrl:operator odrl:eq ;
        odrl:rightOperand dcon:Professional
    ] .
```

**Adalbert**:
```turtle
[   a odrl:Permission ;
    odrl:assignee ex:analyticsTeam ;
    odrl:action adalbert-due:display ;
    odrl:target ex:marketDataFeed ;
    odrl:constraint [
        a odrl:Constraint ;
        odrl:leftOperand adalbert-due:recipientType ;
        odrl:operator odrl:eq ;
        odrl:rightOperand adalbert-due:professional
    ]
] .
```

---

## 5. Property Mapping Reference

### 5.1 Policy Properties

| DCON Property | Adalbert Property | Notes |
|---------------|-------------------|-------|
| `dcon:provider` | `odrl:assigner` | Data provider |
| `dcon:subscriber` | `odrl:assignee` | Data consumer |
| `odrl:target` | `odrl:target` | Policy-level asset |
| `dcon:contractState` | `adalbert:state` | Unified lifecycle state |
| `dcon:effectiveDate` | `adalbert:effectiveDate` | Start date |
| `dcon:expirationDate` | `adalbert:expirationDate` | End date |
| `dcon:subscribesTo` | `adalbert:subscribesTo` | Contract reference |

### 5.2 Promise/Duty Properties

| DCON Property | Adalbert Property | Notes |
|---------------|-------------------|-------|
| `dcon:promisor` | `odrl:assignee` | Who bears the duty |
| `dcon:promisee` | (implicit) | The other party |
| `dcon:promiseState` | `adalbert:state` | Unified state (different machines) |
| `dcon:promisedDeliveryTime` | `adalbert:deadline` | As time or duration |
| `dcon:notificationLeadTime` | `adalbert:deadline` | As duration |

### 5.3 Constraint Properties

| DCON Property | Adalbert Property | Notes |
|---------------|-------------------|-------|
| `odrl:leftOperand` | `odrl:leftOperand` | Same |
| `odrl:operator` | `odrl:operator` | Same |
| `odrl:rightOperand` | `odrl:rightOperand` | Same |
| `odrl:constraint` | `odrl:constraint` | Same |

---

## 6. What Adalbert Does NOT Model

| DCON Feature | Adalbert Status | Rationale |
|--------------|-----------------|-----------|
| Promise Theory semantics | Duties only | Simpler; RL2 will have full Promise support |
| Promisor/promisee distinction | `odrl:assignee` only | Implicit from agreement parties |
| dcat:Distribution | Use DCAT directly | Not policy semantics |
| Subscription billing | Out of scope | Operational concern |
| Multi-party contracts | Two parties only | Bilateral model |
| Draft/Published states | Not modeled | Pre-normative workflow metadata |

---

## 7. SPARQL Transformation

For bulk migration, this CONSTRUCT query transforms DCON to Adalbert:

```sparql
PREFIX dcon: <https://vocabulary.bigbank/dcon/>
PREFIX adalbert: <https://vocabulary.bigbank/adalbert/>
PREFIX odrl: <http://www.w3.org/ns/odrl/2/>

CONSTRUCT {
    ?contract a adalbert:DataContract ;
        odrl:assigner ?provider ;
        odrl:target ?target .

    ?subscription a adalbert:Subscription ;
        adalbert:subscribesTo ?contract ;
        odrl:assigner ?provider ;
        odrl:assignee ?subscriber ;
        adalbert:effectiveDate ?effective ;
        adalbert:expirationDate ?expiry .
}
WHERE {
    ?contract a dcon:DataContract ;
        dcon:provider ?provider .
    OPTIONAL { ?contract odrl:target ?target }

    OPTIONAL {
        ?subscription a dcon:DataContractSubscription ;
            dcon:subscribesTo ?contract ;
            dcon:subscriber ?subscriber .
        OPTIONAL { ?subscription dcon:effectiveDate ?effective }
        OPTIONAL { ?subscription dcon:expirationDate ?expiry }
    }
}
```

---

## 8. Validation

After migration, validate with Adalbert SHACL shapes:

```bash
shacl validate --shapes adalbert-shacl.ttl --data migrated.ttl
```

Expected: No violations for well-formed DCON contracts.
