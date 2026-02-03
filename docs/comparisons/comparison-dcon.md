# Adalbert ↔ DCON Integration Guide

**Purpose**: Complete mapping between DCON ontology and Adalbert, with migration examples.

---

## Executive Summary

| DCON Concept | Adalbert Equivalent | Relationship |
|--------------|---------------------|--------------|
| `dcon:DataContract` | `adalbert-dc:DataContract` | `skos:closeMatch` |
| `dcon:DataContractSubscription` | `adalbert-dc:Subscription` | `skos:closeMatch` |
| `dcon:Promise` | `adalbert:Duty` | Partial (see §3) |
| `dcon:Permission` | `adalbert:Privilege` | `skos:closeMatch` |
| Promise states | Duty states | Different state machines |

**Key difference**: DCON has 3 duty states (Pending→Fulfilled/Violated), Adalbert has 4 (Pending→Active→Fulfilled/Violated). The extra `Active` state represents "condition satisfied, awaiting performance."

---

## 1. Policy Type Mapping

### 1.1 DataContract → adalbert-dc:DataContract

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
@prefix adalbert: <https://vocabulary.bigbank/adalbert/> .
@prefix adalbert-dc: <https://vocabulary.bigbank/adalbert/contracts/> .

ex:marketDataContract a adalbert-dc:DataContract ;
    adalbert:grantor ex:dataTeam ;
    adalbert:target ex:marketDataFeed ;
    adalbert-dc:contractState adalbert-dc:Published ;
    
    adalbert:clause [
        a adalbert:Privilege ;
        adalbert:subject adalbert:currentAgent ;  # Any subscriber
        adalbert:action adalbert-md:display ;
        adalbert:object ex:marketDataFeed
    ] ;
    
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:dataTeam ;  # Provider duty
        adalbert:action adalbert-md:notify ;
        adalbert:object ex:schemaChanges ;
        adalbert:deadline "P7D"^^xsd:duration
    ] .
```

### 1.2 DataContractSubscription → adalbert-dc:Subscription

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
ex:analyticsSubscription a adalbert-dc:Subscription ;
    adalbert-dc:subscribesTo ex:marketDataContract ;
    adalbert:grantor ex:dataTeam ;      # Inherited from contract
    adalbert:grantee ex:analyticsTeam ;
    adalbert-dc:effectiveDate "2026-01-15T00:00:00Z"^^xsd:dateTime ;
    
    # Clauses copied/instantiated from contract
    adalbert:clause [
        a adalbert:Privilege ;
        adalbert:subject ex:analyticsTeam ;
        adalbert:action adalbert-md:display ;
        adalbert:object ex:marketDataFeed
    ] ;
    
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:dataTeam ;
        adalbert:action adalbert-md:notify ;
        adalbert:object ex:schemaChanges ;
        adalbert:deadline "P7D"^^xsd:duration ;
        adalbert:dutyState adalbert:Pending
    ] ;
    
    # Consumer duty (not in contract template)
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:analyticsTeam ;
        adalbert:action adalbert-md:report ;
        adalbert:object ex:usageStatistics ;
        adalbert:deadline "P30D"^^xsd:duration ;
        adalbert:dutyState adalbert:Pending
    ] .
```

---

## 2. Contract State Mapping

| DCON State | Adalbert State | Notes |
|------------|----------------|-------|
| `dcon:Draft` | `adalbert-dc:Draft` | Under development |
| `dcon:Published` | `adalbert-dc:Published` | Available for subscription |
| `dcon:Active` | `adalbert-dc:Active` | Has active subscriptions |
| `dcon:Retired` | `adalbert-dc:Retired` | No new subscriptions |
| `dcon:Cancelled` | `adalbert-dc:Cancelled` | Cancelled before publish |

These are `skos:closeMatch` — same lifecycle, possibly different transition rules.

---

## 3. Promise → Duty Translation

DCON distinguishes Promises (ought-to-be commitments) from Duties (ought-to-do obligations). Adalbert collapses this distinction: **all obligations are Duties**.

### 3.1 State Machine Difference

**DCON Promise States** (3 states):
```
Pending ──────────────> Fulfilled
    │                       
    └──────────────────> Violated
```

**Adalbert Duty States** (4 states):
```
Pending ──[condition]──> Active ──[action]──> Fulfilled
                           │
                           └──[deadline]──> Violated
```

**Mapping**:
- DCON `Pending` ≈ Adalbert `Pending` OR `Active` (condition may not be explicit in DCON)
- DCON `Fulfilled` = Adalbert `Fulfilled`
- DCON `Violated` = Adalbert `Violated`

### 3.2 Provider Promise Types

**DCON ProviderTimelinessPromise**:
```turtle
ex:timelinessPromise a dcon:ProviderTimelinessPromise ;
    dcon:promisor ex:dataTeam ;
    dcon:promisee ex:analyticsTeam ;
    dcon:promisedDeliveryTime "06:00:00"^^xsd:time ;
    dcon:promiseState dcon:Pending .
```

**Adalbert equivalent**:
```turtle
[   a adalbert:Duty ;
    adalbert:subject ex:dataTeam ;
    adalbert:action adalbert-md:deliver ;
    adalbert:object ex:dailyData ;
    adalbert:condition [
        a adalbert:AtomicConstraint ;
        adalbert:leftOperand adalbert:currentDateTime ;
        adalbert:constraintOperator adalbert:gte ;
        adalbert:rightOperand "06:00:00"^^xsd:time
    ] ;
    adalbert:deadline "07:00:00"^^xsd:time ;  # 1 hour grace
    adalbert:dutyState adalbert:Pending
] .
```

**DCON ProviderSchemaPromise**:
```turtle
ex:schemaPromise a dcon:ProviderSchemaPromise ;
    dcon:promisor ex:dataTeam ;
    dcon:promisedSchema ex:marketDataSchema ;
    dcon:promiseState dcon:Pending .
```

**Adalbert equivalent**:
```turtle
[   a adalbert:Duty ;
    adalbert:subject ex:dataTeam ;
    adalbert:action adalbert-gov:conformTo ;
    adalbert:object ex:marketDataSchema ;
    adalbert:dutyState adalbert:Active  # Always active once subscribed
] .
```

**DCON ProviderChangeNotificationPromise**:
```turtle
ex:notifyPromise a dcon:ProviderChangeNotificationPromise ;
    dcon:promisor ex:dataTeam ;
    dcon:notificationLeadTime "P7D"^^xsd:duration ;
    dcon:promiseState dcon:Pending .
```

**Adalbert equivalent**:
```turtle
[   a adalbert:Duty ;
    adalbert:subject ex:dataTeam ;
    adalbert:action adalbert-md:notify ;
    adalbert:object ex:schemaChanges ;
    adalbert:condition [
        a adalbert:AtomicConstraint ;
        adalbert:leftOperand adalbert-dc:schemaChangeDetected ;
        adalbert:constraintOperator adalbert:eq ;
        adalbert:rightOperand true
    ] ;
    adalbert:deadline "P7D"^^xsd:duration ;  # 7 days from detection
    adalbert:dutyState adalbert:Pending
] .
```

### 3.3 Consumer Promise Types

**DCON ConsumerUsageReportingPromise**:
```turtle
ex:reportingPromise a dcon:ConsumerUsageReportingPromise ;
    dcon:promisor ex:analyticsTeam ;
    dcon:promisee ex:dataTeam ;
    dcon:reportingFrequency "P1M"^^xsd:duration ;
    dcon:promiseState dcon:Pending .
```

**Adalbert equivalent**:
```turtle
[   a adalbert:Duty ;
    adalbert:subject ex:analyticsTeam ;
    adalbert:action adalbert-md:report ;
    adalbert:object ex:usageStatistics ;
    adalbert:deadline "P30D"^^xsd:duration ;  # Monthly
    adalbert:dutyState adalbert:Pending
] .
```

---

## 4. Permission Mapping

Straightforward: `odrl:Permission` → `adalbert:Privilege`.

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
[   a adalbert:Privilege ;
    adalbert:subject ex:analyticsTeam ;
    adalbert:action adalbert-md:display ;
    adalbert:object ex:marketDataFeed ;
    adalbert:condition [
        a adalbert:AtomicConstraint ;
        adalbert:leftOperand adalbert-md:recipientType ;
        adalbert:constraintOperator adalbert:eq ;
        adalbert:rightOperand adalbert-md:professional
    ]
] .
```

---

## 5. Property Mapping Reference

### 5.1 Policy Properties

| DCON Property | Adalbert Property | Notes |
|---------------|-------------------|-------|
| `dcon:provider` | `adalbert:grantor` | Data provider |
| `dcon:subscriber` | `adalbert:grantee` | Data consumer |
| `odrl:target` | `adalbert:target` | Policy-level asset |
| `dcon:contractState` | `adalbert-dc:contractState` | Lifecycle state |
| `dcon:effectiveDate` | `adalbert-dc:effectiveDate` | Start date |
| `dcon:expirationDate` | `adalbert-dc:expirationDate` | End date |
| `dcon:subscribesTo` | `adalbert-dc:subscribesTo` | Contract reference |

### 5.2 Promise/Duty Properties

| DCON Property | Adalbert Property | Notes |
|---------------|-------------------|-------|
| `dcon:promisor` | `adalbert:subject` | Who bears the duty |
| `dcon:promisee` | (implicit) | The other party |
| `dcon:promiseState` | `adalbert:dutyState` | Different state machines |
| `dcon:promisedDeliveryTime` | `adalbert:deadline` | As time or duration |
| `dcon:notificationLeadTime` | `adalbert:deadline` | As duration |

### 5.3 Constraint Properties

| DCON Property | Adalbert Property | Notes |
|---------------|-------------------|-------|
| `odrl:leftOperand` | `adalbert:leftOperand` | Same |
| `odrl:operator` | `adalbert:constraintOperator` | Same operators |
| `odrl:rightOperand` | `adalbert:rightOperand` | Same |
| `odrl:constraint` | `adalbert:condition` | Renamed |

---

## 6. Complete Migration Example

### 6.1 DCON Source

```turtle
@prefix dcon: <https://vocabulary.bigbank/dcon/> .
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .

# Data Contract (Offer)
ex:equityFeedContract a dcon:DataContract ;
    rdfs:label "Equity Market Data Feed Contract" ;
    dcon:provider ex:marketDataTeam ;
    odrl:target ex:equityPrices ;
    dcon:contractState dcon:Published ;
    
    # Provider promises
    dcon:hasPromise [
        a dcon:ProviderTimelinessPromise ;
        dcon:promisor ex:marketDataTeam ;
        dcon:promisedDeliveryTime "06:00:00"^^xsd:time ;
        dcon:promiseState dcon:Pending
    ] ;
    
    dcon:hasPromise [
        a dcon:ProviderChangeNotificationPromise ;
        dcon:promisor ex:marketDataTeam ;
        dcon:notificationLeadTime "P14D"^^xsd:duration ;
        dcon:promiseState dcon:Pending
    ] ;
    
    # Permissions
    odrl:permission [
        odrl:action odrl:display ;
        odrl:target ex:equityPrices ;
        odrl:constraint [
            odrl:leftOperand dcon:timeliness ;
            odrl:operator odrl:eq ;
            odrl:rightOperand dcon:Realtime
        ]
    ] ;
    
    odrl:permission [
        odrl:action dcon:derive ;
        odrl:target ex:equityPrices ;
        odrl:constraint [
            odrl:leftOperand dcon:derivationType ;
            odrl:operator odrl:eq ;
            odrl:rightOperand dcon:NonSubstitutive
        ]
    ] ;
    
    # Prohibition
    odrl:prohibition [
        odrl:action odrl:distribute ;
        odrl:target ex:equityPrices
    ] .

# Subscription (Agreement)
ex:analyticsSubscription a dcon:DataContractSubscription ;
    dcon:subscribesTo ex:equityFeedContract ;
    dcon:subscriber ex:quantTeam ;
    dcon:subscriptionState dcon:Active ;
    dcon:effectiveDate "2026-01-01T00:00:00Z"^^xsd:dateTime ;
    dcon:expirationDate "2026-12-31T23:59:59Z"^^xsd:dateTime ;
    
    # Consumer promise
    dcon:hasPromise [
        a dcon:ConsumerUsageReportingPromise ;
        dcon:promisor ex:quantTeam ;
        dcon:promisee ex:marketDataTeam ;
        dcon:reportingFrequency "P1M"^^xsd:duration ;
        dcon:promiseState dcon:Pending
    ] .
```

### 6.2 Adalbert Target

```turtle
@prefix adalbert:     <https://vocabulary.bigbank/adalbert/> .
@prefix adalbert-dc:  <https://vocabulary.bigbank/adalbert/contracts/> .
@prefix adalbert-md:  <https://vocabulary.bigbank/adalbert/market-data/> .
@prefix adalbert-gov: <https://vocabulary.bigbank/adalbert/governance/> .
@prefix xsd:          <http://www.w3.org/2001/XMLSchema#> .

# Data Contract (Offer)
ex:equityFeedContract a adalbert-dc:DataContract ;
    rdfs:label "Equity Market Data Feed Contract" ;
    adalbert:grantor ex:marketDataTeam ;
    adalbert:target ex:equityPrices ;
    adalbert-dc:contractState adalbert-dc:Published ;
    
    # Provider duty: timeliness
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:marketDataTeam ;
        adalbert:action adalbert-md:deliver ;
        adalbert:object ex:equityPrices ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand adalbert:currentDateTime ;
            adalbert:constraintOperator adalbert:gte ;
            adalbert:rightOperand "06:00:00"^^xsd:time
        ] ;
        adalbert:deadline "07:00:00"^^xsd:time
    ] ;
    
    # Provider duty: change notification
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:marketDataTeam ;
        adalbert:action adalbert-md:notify ;
        adalbert:object ex:schemaChanges ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand adalbert-dc:schemaChangeDetected ;
            adalbert:constraintOperator adalbert:eq ;
            adalbert:rightOperand true
        ] ;
        adalbert:deadline "P14D"^^xsd:duration
    ] ;
    
    # Privilege: real-time display
    adalbert:clause [
        a adalbert:Privilege ;
        adalbert:subject adalbert:currentAgent ;
        adalbert:action adalbert-md:display ;
        adalbert:object ex:equityPrices ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand adalbert-md:timeliness ;
            adalbert:constraintOperator adalbert:eq ;
            adalbert:rightOperand adalbert-md:realtime
        ]
    ] ;
    
    # Privilege: non-substitutive derivation
    adalbert:clause [
        a adalbert:Privilege ;
        adalbert:subject adalbert:currentAgent ;
        adalbert:action adalbert-md:derive ;
        adalbert:object ex:equityPrices ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand adalbert-md:derivationType ;
            adalbert:constraintOperator adalbert:eq ;
            adalbert:rightOperand adalbert-md:nonSubstitutive
        ]
    ] ;
    
    # Prohibition: no distribution
    adalbert:clause [
        a adalbert:Prohibition ;
        adalbert:subject adalbert:currentAgent ;
        adalbert:action adalbert-gov:distribute ;
        adalbert:object ex:equityPrices
    ] .


# Subscription (Agreement)
ex:analyticsSubscription a adalbert-dc:Subscription ;
    rdfs:label "Quant Team Equity Feed Subscription" ;
    adalbert-dc:subscribesTo ex:equityFeedContract ;
    adalbert:grantor ex:marketDataTeam ;
    adalbert:grantee ex:quantTeam ;
    adalbert-dc:effectiveDate "2026-01-01T00:00:00Z"^^xsd:dateTime ;
    adalbert-dc:expirationDate "2026-12-31T23:59:59Z"^^xsd:dateTime ;
    
    # === GRANTOR DUTIES (Provider SLA) ===
    
    adalbert:clause [
        a adalbert:Duty ;
        rdfs:label "Daily delivery by 7am" ;
        adalbert:subject ex:marketDataTeam ;
        adalbert:action adalbert-md:deliver ;
        adalbert:object ex:equityPrices ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand adalbert:currentDateTime ;
            adalbert:constraintOperator adalbert:gte ;
            adalbert:rightOperand "06:00:00"^^xsd:time
        ] ;
        adalbert:deadline "07:00:00"^^xsd:time ;
        adalbert:dutyState adalbert:Pending
    ] ;
    
    adalbert:clause [
        a adalbert:Duty ;
        rdfs:label "14-day schema change notice" ;
        adalbert:subject ex:marketDataTeam ;
        adalbert:action adalbert-md:notify ;
        adalbert:object ex:schemaChanges ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand adalbert-dc:schemaChangeDetected ;
            adalbert:constraintOperator adalbert:eq ;
            adalbert:rightOperand true
        ] ;
        adalbert:deadline "P14D"^^xsd:duration ;
        adalbert:dutyState adalbert:Pending
    ] ;
    
    # === GRANTEE PRIVILEGES (Consumer Rights) ===
    
    adalbert:clause [
        a adalbert:Privilege ;
        rdfs:label "Real-time display" ;
        adalbert:subject ex:quantTeam ;
        adalbert:action adalbert-md:display ;
        adalbert:object ex:equityPrices ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand adalbert-md:timeliness ;
            adalbert:constraintOperator adalbert:eq ;
            adalbert:rightOperand adalbert-md:realtime
        ]
    ] ;
    
    adalbert:clause [
        a adalbert:Privilege ;
        rdfs:label "Non-substitutive derivation" ;
        adalbert:subject ex:quantTeam ;
        adalbert:action adalbert-md:derive ;
        adalbert:object ex:equityPrices ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand adalbert-md:derivationType ;
            adalbert:constraintOperator adalbert:eq ;
            adalbert:rightOperand adalbert-md:nonSubstitutive
        ]
    ] ;
    
    # === GRANTEE DUTIES (Consumer Obligations) ===
    
    adalbert:clause [
        a adalbert:Duty ;
        rdfs:label "Monthly usage reporting" ;
        adalbert:subject ex:quantTeam ;
        adalbert:action adalbert-md:report ;
        adalbert:object ex:usageStatistics ;
        adalbert:deadline "P30D"^^xsd:duration ;
        adalbert:dutyState adalbert:Pending
    ] ;
    
    # === GRANTEE PROHIBITIONS ===
    
    adalbert:clause [
        a adalbert:Prohibition ;
        rdfs:label "No redistribution" ;
        adalbert:subject ex:quantTeam ;
        adalbert:action adalbert-gov:distribute ;
        adalbert:object ex:equityPrices
    ] .
```

---

## 7. What Adalbert Does NOT Model

| DCON Feature | Adalbert Status | Rationale |
|--------------|-----------------|-----------|
| Promise Theory semantics | Duties only | Simpler; RL2 will have full Promise support |
| Promisor/promisee distinction | Subject only | Implicit from agreement parties |
| dcat:Distribution | Use DCAT directly | Not policy semantics |
| Subscription billing | Out of scope | Operational concern |
| Multi-party contracts | Two parties only | Bilateral model |

---

## 8. SPARQL Transformation

For bulk migration, this CONSTRUCT query transforms DCON to Adalbert:

```sparql
PREFIX dcon: <https://vocabulary.bigbank/dcon/>
PREFIX adalbert: <https://vocabulary.bigbank/adalbert/>
PREFIX adalbert-dc: <https://vocabulary.bigbank/adalbert/contracts/>

CONSTRUCT {
    ?contract a adalbert-dc:DataContract ;
        adalbert:grantor ?provider ;
        adalbert:target ?target ;
        adalbert-dc:contractState ?state .
    
    ?subscription a adalbert-dc:Subscription ;
        adalbert-dc:subscribesTo ?contract ;
        adalbert:grantor ?provider ;
        adalbert:grantee ?subscriber ;
        adalbert-dc:effectiveDate ?effective ;
        adalbert-dc:expirationDate ?expiry .
}
WHERE {
    ?contract a dcon:DataContract ;
        dcon:provider ?provider .
    OPTIONAL { ?contract odrl:target ?target }
    OPTIONAL { ?contract dcon:contractState ?dconState }
    BIND(IRI(REPLACE(STR(?dconState), "dcon:", "adalbert-dc:")) AS ?state)
    
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

## 9. Validation

After migration, validate with Adalbert SHACL shapes:

```bash
shacl validate --shapes adalbert-shacl.ttl --data migrated.ttl
```

Expected: No violations for well-formed DCON contracts.
