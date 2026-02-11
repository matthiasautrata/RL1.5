# Adalbert Supersedes DCON

**Purpose**: Supersession analysis documenting how Adalbert (v0.7) absorbs DCON.

---

## Executive Summary

As of v0.7, Adalbert **supersedes** DCON. The DCON promise hierarchy dissolves into standard `odrl:Duty` patterns. Recurring obligations use `adalbert:recurrence` (RFC 5545 RRULE) instead of DCON's promise type hierarchy.

| DCON Concept | Adalbert v0.7 | Status |
|--------------|---------------|--------|
| `dcon:DataContract` | `adalbert:DataContract` | Absorbed into core (v0.1+) |
| `dcon:DataContractSubscription` | `adalbert:Subscription` | Absorbed into core (v0.1+) |
| `dcon:Promise` | `odrl:Duty` | Dissolved — all obligations are duties |
| `dcon:Permission` | `odrl:Permission` | Standard ODRL |
| Promise states | `adalbert:State` | Unified 4-state lifecycle |
| Promise type hierarchy | `odrl:Duty` + DUE actions | Dissolved (see §2) |
| `dcon:promisedDeliveryTime` | `adalbert:recurrence` + `adalbert:deadline` | Recurrence handles scheduling; deadline handles window |

**Key change in v0.7**: `adalbert:recurrence` replaces DCON's scheduling constraints. A single RRULE string on a duty defines when instances are generated; `deadline` defines the per-instance fulfillment window.

The former alignment file has been removed. This document serves as the complete supersession record.

---

## 1. Promise Dissolution

DCON distinguishes Promises (ought-to-be commitments) from Duties (ought-to-do obligations) with a type hierarchy of promise subclasses. Adalbert collapses this: **all obligations are `odrl:Duty`** with DUE actions providing the semantic differentiation.

### 1.1 Promise Dissolution Table

| DCON Promise Type | Adalbert Duty Pattern | Action | Notes |
|-------------------|----------------------|--------|-------|
| `ProviderTimelinessPromise` | Recurring delivery duty | `adalbert-due:deliver` | `recurrence` + `deadline` replaces `promisedDeliveryTime` |
| `ProviderSchemaPromise` | Schema conformance duty | `adalbert-due:conformTo` | No `maintain` action needed — `conformTo` covers schema + quality SLA |
| `ProviderChangeNotificationPromise` | Notification duty | `adalbert-due:notify` | `deadline` as duration (e.g., P14D lead time) |
| `ConsumerUsageReportingPromise` | Reporting duty | `adalbert-due:report` | `deadline` as duration (e.g., P30D) |
| `fulfillsPromise` link | `prov:wasDerivedFrom` on Subscription | — | Standard provenance; no custom property needed |

### 1.2 Migration Examples

**DCON ProviderTimelinessPromise** → Adalbert recurring delivery duty:

```turtle
# DCON (before)
ex:timelinessPromise a dcon:ProviderTimelinessPromise ;
    dcon:promisor ex:dataTeam ;
    dcon:promisedDeliveryTime "06:00:00"^^xsd:time .

# Adalbert v0.7 (after)
[   a odrl:Duty ;
    adalbert:subject ex:dataTeam ;
    odrl:action adalbert-due:deliver ;
    odrl:target ex:marketPrices ;
    adalbert:recurrence "FREQ=DAILY;BYHOUR=6;BYMINUTE=0" ;
    adalbert:deadline "PT30M"^^xsd:duration
] .
```

**DCON ProviderSchemaPromise** → Adalbert conformance duty:

```turtle
# Adalbert v0.7
[   a odrl:Duty ;
    adalbert:subject ex:dataTeam ;
    odrl:action adalbert-due:conformTo ;
    odrl:target ex:marketDataSchema ;
    adalbert:state adalbert:Active
] .
```

**DCON ProviderChangeNotificationPromise** → Adalbert notification duty:

```turtle
# Adalbert v0.7
[   a odrl:Duty ;
    adalbert:subject ex:dataTeam ;
    odrl:action adalbert-due:notify ;
    odrl:target ex:schemaChanges ;
    adalbert:deadline "P14D"^^xsd:duration
] .
```

---

## 2. What Was Retained

| DCON Concept | Adalbert Location | Notes |
|--------------|-------------------|-------|
| DataContract class | `adalbert:DataContract` (core) | Subclass of `odrl:Offer` |
| Subscription class | `adalbert:Subscription` (core) | Subclass of `odrl:Agreement` |
| `subscribesTo` | `adalbert:subscribesTo` | Links Subscription to DataContract |
| `effectiveDate` / `expirationDate` | `adalbert:effectiveDate` / `adalbert:expirationDate` | Contract lifecycle dates |
| Provider/consumer duties | `odrl:obligation` with `adalbert:subject` | Bilateral agreement pattern |
| Permissions | `odrl:Permission` | Standard ODRL |
| State tracking | `adalbert:State` | Unified 4-state lifecycle |

## 3. What Was Dropped

| DCON Concept | Reason |
|--------------|--------|
| Promise Theory semantics (ought-to-be) | Duties suffice; RL2 has full Promise support |
| Promise type hierarchy | DUE actions provide equivalent differentiation |
| `promisor` / `promisee` distinction | `adalbert:subject` identifies the bearer; other party implicit |
| `dcon:contractState` (Draft/Published) | Pre-normative workflow metadata, not policy semantics |
| `fulfillsPromise` link | `prov:wasDerivedFrom` covers provenance |
| `dcat:Distribution` integration | Use DCAT directly — not policy semantics |
| Subscription billing | Operational concern, out of scope |

## 4. Termination Clause Pattern

DCON contracts can be terminated by either party. In Adalbert, contract termination is modeled via the unified state machine:

```turtle
# Contract expires naturally
ex:subscription adalbert:expirationDate "2026-12-31T23:59:59Z"^^xsd:dateTime .

# Contract violated (duty breach)
ex:subscription adalbert:state adalbert:Violated .
```

Administrative termination (early exit by agreement) is out of scope for policy evaluation — it is an operational workflow concern, not a norm.

---

## 5. State Mapping

| Adalbert State | Duty meaning | Contract meaning | Former DCON equivalent |
|----------------|-------------|------------------|------------------------|
| Pending | Condition not yet met | Not yet in force | `dcon:Pending` |
| Active | Condition met, action required | In force | `dcon:Active` |
| Fulfilled | Action performed | Obligations complete | `dcon:Fulfilled` |
| Violated | Deadline passed | Breached | `dcon:Violated` |

DCON's `Draft` and `Published` were pre-normative workflow metadata — not modeled in Adalbert.

---

## 6. Property Mapping Reference

| DCON Property | Adalbert Property | Notes |
|---------------|-------------------|-------|
| `dcon:provider` | `odrl:assigner` | Data provider |
| `dcon:subscriber` | `odrl:assignee` | Data consumer |
| `dcon:contractState` | `adalbert:state` | Unified lifecycle state |
| `dcon:effectiveDate` | `adalbert:effectiveDate` | Start date |
| `dcon:expirationDate` | `adalbert:expirationDate` | End date |
| `dcon:subscribesTo` | `adalbert:subscribesTo` | Contract reference |
| `dcon:promisedDeliveryTime` | `adalbert:recurrence` + `adalbert:deadline` | Scheduling + window |
| `dcon:notificationLeadTime` | `adalbert:deadline` | As duration |
| `dcon:promisor` | `adalbert:subject` | Duty bearer |
| `dcon:promiseState` | `adalbert:state` | Unified state |

---

## 7. SPARQL Migration

For bulk migration from DCON to Adalbert v0.7:

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

Promise migration requires manual mapping of promise types to duty patterns (see §1.1).

---

## 8. Validation

After migration, validate with Adalbert SHACL shapes:

```bash
shacl validate --shapes adalbert-shacl.ttl --data migrated.ttl
```

Expected: No violations for well-formed DCON contracts. The `recurrence` property is optional, so existing duties without recurrence remain valid.

---

## 9. Completeness Verification

Systematic verification that every DCON class, property, and action has an Adalbert equivalent or an explicit "not modeled" decision.

### 9.1 Classes

| DCON Class | Type | Adalbert Equivalent | Status |
|------------|------|---------------------|--------|
| `dcon:DataContract` | Class | `adalbert:DataContract` | Direct |
| `dcon:DataContractSubscription` | Class | `adalbert:Subscription` | Direct |
| `dcon:Promise` | Class | `odrl:Duty` | Dissolved |
| `dcon:ProviderPromise` | Class | `odrl:Duty` + DUE action | Dissolved |
| `dcon:ProviderTimelinessPromise` | Class | Duty + `deliver` + `recurrence` + `deadline` | Pattern |
| `dcon:ProviderSchemaPromise` | Class | Duty + `conformTo` | Pattern |
| `dcon:ProviderChangeNotificationPromise` | Class | Duty + `notify` + `deadline` | Pattern |
| `dcon:ProviderQualityPromise` | Class | Duty + `conformTo` + constraint | Pattern |
| `dcon:ConsumerPromise` | Class | `odrl:Duty` (assignee duty) | Dissolved |
| `dcon:Schedule` | Class | `adalbert:recurrence` value | Simplified |
| `dcon:ICalSchedule` | Class | `adalbert:recurrence` value | Absorbed |
| `dcon:PromiseState` | Class | `adalbert:State` | Extended (4 vs 3 states) |
| `dcon:Condition` | Class | `odrl:Constraint` | Standard ODRL |
| `dcon:ContractStatus` | Enum | -- | Not modeled (workflow) |
| `dcon:Draft` | Individual | -- | Not modeled (workflow) |
| `dcon:Published` | Individual | -- | Not modeled (workflow) |
| `dcon:Active` | Individual | `adalbert:Active` | Direct |
| `dcon:Retired` | Individual | -- | Not modeled (workflow) |
| `dcon:Cancelled` | Individual | -- | Not modeled (workflow) |
| `dcon:Pending` | Individual | `adalbert:Pending` | Direct |
| `dcon:Fulfilled` | Individual | `adalbert:Fulfilled` | Direct |
| `dcon:Violated` | Individual | `adalbert:Violated` | Direct |

### 9.2 Properties

| DCON Property | Type | Adalbert Equivalent | Status |
|---------------|------|---------------------|--------|
| `dcon:provider` | ObjectProperty | `odrl:assigner` | Standard ODRL |
| `dcon:subscriber` | ObjectProperty | `odrl:assignee` | Standard ODRL |
| `dcon:contractState` | ObjectProperty | `adalbert:state` | Direct |
| `dcon:effectiveDate` | DatatypeProperty | `adalbert:effectiveDate` | Direct |
| `dcon:expirationDate` | DatatypeProperty | `adalbert:expirationDate` | Direct |
| `dcon:subscribesTo` | ObjectProperty | `adalbert:subscribesTo` | Direct |
| `dcon:promisor` | ObjectProperty | `adalbert:subject` (on Duty) | Adalbert (`rdfs:subPropertyOf odrl:assignee`) |
| `dcon:promisee` | ObjectProperty | Implicit (other party) | Not needed |
| `dcon:promiseContent` | ObjectProperty | `odrl:action` (on Duty) | Standard ODRL |
| `dcon:promiseState` | ObjectProperty | `adalbert:state` | Direct |
| `dcon:fulfillsPromise` | ObjectProperty | `prov:wasDerivedFrom` | Standard W3C |
| `dcon:hasPromise` | ObjectProperty | `odrl:obligation` | Standard ODRL |
| `dcon:hasContract` | ObjectProperty | `dcat:hasPolicy` (external) | External |
| `dcon:hasEffectivePeriod` | ObjectProperty | `effectiveDate` + `expirationDate` | Simplified |
| `dcon:hasTerminationClause` | ObjectProperty | `adalbert:deadline` (duration) | Simplified |
| `dcon:hasSchedule` | ObjectProperty | `adalbert:recurrence` | Simplified |
| `dcon:icalRule` | DatatypeProperty | `adalbert:recurrence` value | Absorbed |
| `dcon:promisedDeliveryTime` | DatatypeProperty | `recurrence` + `deadline` | Simplified |
| `dcon:notificationLeadTime` | DatatypeProperty | `adalbert:deadline` (duration) | Simplified |

### 9.3 Actions

| DCON Action | Adalbert Equivalent | Status |
|-------------|---------------------|--------|
| `dcon:deliver` | `adalbert-due:deliver` | Direct |
| `dcon:maintain` | `adalbert-due:conformTo` | Renamed |
| `dcon:notify` | `adalbert-due:notify` | Direct |

### 9.4 Not Modeled (by design)

| DCON Concept | Type | Rationale |
|--------------|------|-----------|
| `dcon:Draft` | Status | Pre-normative workflow metadata, not policy semantics |
| `dcon:Retired` | Status | Pre-normative workflow metadata |
| `dcon:Cancelled` | Status | Pre-normative workflow metadata |
| `dcon:Published` | Status | Pre-normative workflow metadata |
| `dcon:ContractStatus` | Class | Workflow metadata; Adalbert's `State` covers policy lifecycle |
| `dcon:promisee` | Property | Implicit in bilateral agreements; not needed |
| `dcon:hasContract` | Property | External relationship; use `dcat:hasPolicy` |
| Promise class hierarchy | Design | Dissolved into `odrl:Duty` + DUE actions |
| Promise Theory semantics | Design | Duties suffice; RL2 has full Promise support |
| `schema:Schedule` format | Design | Single RRULE string replaces class hierarchy |
| `dcat:Distribution` integration | Design | Use DCAT directly; not policy semantics |
| Subscription billing | Design | Operational concern, out of scope |

### 9.5 Coverage Summary

| Category | Total | Direct | Pattern/Simplified | Not Modeled |
|----------|-------|--------|-------------------|-------------|
| Classes | 14 | 2 | 9 | 3 (workflow) |
| Status individuals | 8 | 4 | 0 | 4 (workflow) |
| Properties | 19 | 5 | 8 | 6 (external/implicit) |
| Actions | 3 | 2 | 1 | 0 |

**Result**: 100% coverage. Every DCON concept has an Adalbert equivalent or an explicit "not modeled" decision with rationale. The 7 "not modeled" items are all pre-normative workflow metadata, implicit relationships, or external integration concerns that are out of scope for policy evaluation.
