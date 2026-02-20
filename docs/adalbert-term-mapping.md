# Adalbert Term Mapping

Maps business language to Adalbert/ODRL vocabulary. Includes complete DCON -> Adalbert migration mapping.

---

## 1. How to Use This Document

This document bridges the gap between business terminology and the technical ODRL/Adalbert vocabulary. Each section covers a category of terms. For each term you will find:

- **Business term**: what people say in conversation
- **Adalbert property**: the RDF property or class to use
- **Used on**: which policy elements this applies to
- **Cardinality**: how many values are allowed
- **Source**: whether the term comes from ODRL or Adalbert
- **DCON equivalent**: for migration reference

For the formal vocabulary reference, see [adalbert-specification.md](adalbert-specification.md). For authoring guidance, see [contracts-guide.md](contracts-guide.md) or [policy-writers-guide.md](policy-writers-guide.md).

---

## 2. Contract Metadata

### Contract Title

**Business term:** `title`, `contract name`, `policy name`
**Adalbert property:** `rdfs:label`
**Used on:** `adalbert:DataContract`, `adalbert:Subscription`, `odrl:Set`
**Cardinality:** 0..1
**Source:** RDFS
**Example:**
```turtle
ex:contract a adalbert:DataContract ;
    rdfs:label "Market Price Data Contract v2" .
```
**DCON equivalent:** `rdfs:label` (same)

### Contract Description

**Business term:** `description`, `summary`
**Adalbert property:** `dcterms:description`
**Used on:** Any policy
**Cardinality:** 0..1
**Source:** Dublin Core

### Contract Version

**Business term:** `version`, `revision`
**Adalbert property:** `prov:wasRevisionOf`
**Used on:** `adalbert:DataContract`
**Cardinality:** 0..1
**Source:** W3C PROV
**Example:**
```turtle
ex:contract-v2 a adalbert:DataContract ;
    prov:wasRevisionOf ex:contract-v1 .
```
**DCON equivalent:** `prov:wasRevisionOf` (same)

### Contract Status

**Business term:** `status`, `state`, `lifecycle state`
**Adalbert property:** `adalbert:state`
**Used on:** `adalbert:DataContract`, `adalbert:Subscription`, `odrl:Duty`
**Cardinality:** 0..1
**Values:** `adalbert:Pending`, `adalbert:Active`, `adalbert:Fulfilled`, `adalbert:Violated`
**Source:** Adalbert
**DCON equivalent:** `dcon:contractState` (partial; DCON also had Draft/Published which are not modeled)

---

## 3. Temporal Properties

### Effective Date

**Business term:** `start date`, `effective date`, `goes live`
**Adalbert property:** `adalbert:effectiveDate`
**Used on:** `adalbert:DataContract`, `adalbert:Subscription`
**Cardinality:** 0..1
**Datatype:** `xsd:dateTime`
**Source:** Adalbert
**Example:**
```turtle
ex:subscription adalbert:effectiveDate "2026-01-15T00:00:00Z"^^xsd:dateTime .
```
**DCON equivalent:** `dcon:effectiveDate`

### Expiration Date

**Business term:** `end date`, `expiry`, `contract end`
**Adalbert property:** `adalbert:expirationDate`
**Used on:** `adalbert:DataContract`, `adalbert:Subscription`
**Cardinality:** 0..1
**Datatype:** `xsd:dateTime`
**Source:** Adalbert
**DCON equivalent:** `dcon:expirationDate`

### Deadline

**Business term:** `due date`, `SLA window`, `fulfillment deadline`
**Adalbert property:** `adalbert:deadline`
**Used on:** `odrl:Duty`
**Cardinality:** 0..1
**Datatype:** `xsd:dateTime` | `xsd:duration`
**Source:** Adalbert
**Example:**
```turtle
# Absolute deadline
adalbert:deadline "2026-12-31T23:59:59Z"^^xsd:dateTime .

# Relative deadline (30 days from activation)
adalbert:deadline "P30D"^^xsd:duration .
```
**DCON equivalent:** `dcon:hasTerminationClause` (partial)

### Recurrence Schedule

**Business term:** `schedule`, `frequency`, `how often`
**Adalbert property:** `adalbert:recurrence`
**Used on:** `odrl:Duty`
**Cardinality:** 0..1
**Datatype:** `xsd:string` (RFC 5545 RRULE)
**Source:** Adalbert
**Example:**
```turtle
adalbert:recurrence "FREQ=DAILY;BYHOUR=6;BYMINUTE=0" .
```
**DCON equivalent:** `dcon:hasSchedule` + `dcon:icalRule` (two properties collapsed into one)

---

## 4. Parties and Responsibility

### Contract Provider

**Business term:** `provider`, `data provider`, `data owner`, `publisher`
**Adalbert property:** `odrl:assigner`
**Used on:** `adalbert:DataContract`, `adalbert:Subscription`
**Cardinality:** 1 (required)
**Source:** ODRL
**Example:**
```turtle
ex:contract a adalbert:DataContract ;
    odrl:assigner ex:dataTeam .
```
**DCON equivalent:** `odrl:assigner` (same)

### Contract Consumer

**Business term:** `consumer`, `subscriber`, `data consumer`, `client`
**Adalbert property:** `odrl:assignee`
**Used on:** `adalbert:Subscription` (required), `odrl:Permission`/`odrl:Duty` (optional)
**Cardinality:** 1 on Subscription; 0..1 on rules
**Source:** ODRL
**Example:**
```turtle
ex:subscription a adalbert:Subscription ;
    odrl:assignee ex:analyticsTeam .
```
**DCON equivalent:** `dcon:subscriber` -> `odrl:assignee`

### Duty Holder

**Business term:** `responsible party`, `obligee`, `who must do it`
**Adalbert property:** `adalbert:subject` (on `odrl:Duty`)
**Used on:** `odrl:Duty`
**Cardinality:** 0..1
**Source:** Adalbert (`rdfs:subPropertyOf odrl:assignee`)
**Note:** In a DataContract (Offer), provider duties have `adalbert:subject` set to the provider. Consumer duties omit `adalbert:subject` — it is filled in when the Subscription is created.
**DCON equivalent:** `dcon:promisor`

### Party Hierarchy

**Business term:** `team membership`, `department`, `division`
**Adalbert property:** `adalbert:memberOf`
**Used on:** `odrl:Party`
**Cardinality:** 0..*
**Source:** Adalbert
**Example:**
```turtle
ex:analyst a odrl:Party ;
    adalbert:memberOf ex:analyticsTeam .
ex:analyticsTeam a odrl:Party ;
    adalbert:memberOf ex:tradingDivision .
```

---

## 5. Assets

### Target Asset

**Business term:** `data`, `dataset`, `covered data`, `target`
**Adalbert property:** `odrl:target`
**Used on:** Policies, `odrl:Permission`, `odrl:Duty`, `odrl:Prohibition`
**Cardinality:** 1..* on policies; 1 on rules
**Source:** ODRL
**Example:**
```turtle
ex:contract odrl:target ex:marketPrices .
```
**DCON equivalent:** `odrl:target` (same)

### Asset Hierarchy

**Business term:** `part of`, `contained in`, `sub-asset`
**Adalbert property:** `adalbert:partOf`
**Used on:** `odrl:Asset`
**Cardinality:** 0..*
**Source:** Adalbert
**Example:**
```turtle
ex:priceTable a odrl:Asset ;
    adalbert:partOf ex:marketDataSchema .
ex:marketDataSchema a odrl:Asset ;
    adalbert:partOf ex:marketDataLake .
```

---

## 6. Obligations

### Delivery SLA

**Business term:** `data delivery`, `SLA`, `timeliness guarantee`
**Adalbert pattern:** `odrl:Duty` with `adalbert-due:deliver` + `adalbert:recurrence` + `adalbert:deadline`
**Example:**
```turtle
odrl:obligation [
    a odrl:Duty ;
    adalbert:subject ex:dataTeam ;
    odrl:action adalbert-due:deliver ;
    odrl:target ex:marketPrices ;
    adalbert:recurrence "FREQ=DAILY;BYHOUR=6;BYMINUTE=0" ;
    adalbert:deadline "PT30M"^^xsd:duration
] .
```
**DCON equivalent:** `dcon:ProviderTimelinessPromise`

### Schema Conformance

**Business term:** `schema guarantee`, `data quality SLA`, `format compliance`
**Adalbert pattern:** `odrl:Duty` with `adalbert-due:conformTo`
**Example:**
```turtle
odrl:obligation [
    a odrl:Duty ;
    adalbert:subject ex:dataTeam ;
    odrl:action adalbert-due:conformTo ;
    odrl:target ex:marketDataSchema
] .
```
**DCON equivalent:** `dcon:ProviderSchemaPromise`

### Quality SLA

**Business term:** `quality guarantee`, `accuracy SLA`, `data quality commitment`
**Adalbert pattern:** `odrl:Duty` with `adalbert-due:conformTo` + `odrl:constraint`
**Example:**
```turtle
odrl:obligation [
    a odrl:Duty ;
    adalbert:subject ex:dataTeam ;
    odrl:action adalbert-due:conformTo ;
    odrl:target ex:riskMetricsSchema ;
    odrl:constraint [
        a odrl:Constraint ;
        odrl:leftOperand adalbert-due:timeliness ;
        odrl:operator odrl:eq ;
        odrl:rightOperand adalbert-due:realtime
    ]
] .
```
**DCON equivalent:** `dcon:ProviderQualityPromise`

### Change Notification

**Business term:** `notification`, `advance notice`, `change alert`
**Adalbert pattern:** `odrl:Duty` with `adalbert-due:notify` + `adalbert:deadline`
**Example:**
```turtle
odrl:obligation [
    a odrl:Duty ;
    adalbert:subject ex:dataTeam ;
    odrl:action adalbert-due:notify ;
    odrl:target ex:schemaChanges ;
    adalbert:deadline "P14D"^^xsd:duration
] .
```
**DCON equivalent:** `dcon:ProviderChangeNotificationPromise`

### Usage Reporting

**Business term:** `usage report`, `consumption report`
**Adalbert pattern:** `odrl:Duty` with `adalbert-due:report` + `adalbert:deadline`
**Example:**
```turtle
odrl:obligation [
    a odrl:Duty ;
    odrl:action adalbert-due:report ;
    odrl:target ex:usageStats ;
    adalbert:deadline "P30D"^^xsd:duration
] .
```
**DCON equivalent:** `dcon:ConsumerUsageReportingPromise` (implied)

---

## 7. Rights and Restrictions

### Display Permission

**Business term:** `can view`, `display rights`, `screen display`
**Adalbert property:** `odrl:display` (action on `odrl:Permission`)
**DCON equivalent:** `odrl:display` (same)

### Non-Display Permission

**Business term:** `algorithmic use`, `automated use`, `programmatic access`
**Adalbert property:** `adalbert-due:nonDisplay` (action on `odrl:Permission`)
**Note:** Distinct from `odrl:display`. Covers models, automation, calculations.

### Derivation Permission

**Business term:** `can create derived data`, `derivation rights`
**Adalbert property:** `odrl:derive` (action on `odrl:Permission`)
**DCON equivalent:** `odrl:derive` (same)

### Distribution Prohibition

**Business term:** `no redistribution`, `no sharing outside`
**Adalbert property:** `odrl:distribute` (action on `odrl:Prohibition`)
**DCON equivalent:** Prohibition with `odrl:distribute`

### Constrained Permission

**Business term:** `can do X if Y`, `conditional access`
**Adalbert pattern:** Permission with `odrl:constraint`
**Example:**
```turtle
odrl:permission [
    a odrl:Permission ;
    odrl:action odrl:read ;
    odrl:target ex:data ;
    odrl:constraint [
        a odrl:Constraint ;
        odrl:leftOperand odrl:purpose ;
        odrl:operator odrl:eq ;
        odrl:rightOperand adalbert-due:analytics
    ]
] .
```

---

## 8. State and Lifecycle

### Duty States

| State | Business meaning | Adalbert value |
|-------|-----------------|----------------|
| Not started | Condition not yet met | `adalbert:Pending` |
| In progress | Action required, deadline ticking | `adalbert:Active` |
| Done | Successfully completed | `adalbert:Fulfilled` |
| Breached | Deadline passed without completion | `adalbert:Violated` |

### Contract States

| State | Business meaning | Adalbert value |
|-------|-----------------|----------------|
| Not yet active | Effective date not reached | `adalbert:Pending` |
| Live | In force, duties being tracked | `adalbert:Active` |
| Complete | All obligations met | `adalbert:Fulfilled` |
| Breached | Material obligation violated | `adalbert:Violated` |

**Note:** DCON's `Draft` and `Published` are pre-normative workflow metadata. Adalbert does not model them — they belong to the authoring system, not policy evaluation.

---

## 9. Quick Reference Table

| Business Term | Adalbert Property / Pattern | Source |
|--------------|---------------------------|--------|
| Contract title | `rdfs:label` | RDFS |
| Provider | `odrl:assigner` | ODRL |
| Consumer | `odrl:assignee` | ODRL |
| Covered data | `odrl:target` | ODRL |
| Start date | `adalbert:effectiveDate` | Adalbert |
| End date | `adalbert:expirationDate` | Adalbert |
| Status | `adalbert:state` | Adalbert |
| Delivery SLA | Duty + `deliver` + `recurrence` + `deadline` | Adalbert + DUE |
| Schema guarantee | Duty + `conformTo` | DUE |
| Quality SLA | Duty + `conformTo` + constraint | DUE |
| Change notice | Duty + `notify` + `deadline` | DUE |
| Usage report | Duty + `report` + `deadline` | DUE |
| View rights | Permission + `display` | ODRL |
| Algo use | Permission + `nonDisplay` | DUE |
| Derivation | Permission + `derive` | ODRL |
| No sharing | Prohibition + `distribute` | ODRL |
| Team membership | `adalbert:memberOf` | Adalbert |
| Data hierarchy | `adalbert:partOf` | Adalbert |
| Version chain | `prov:wasRevisionOf` | W3C PROV |
| Schedule | `adalbert:recurrence` (RRULE) | Adalbert |
| Deadline | `adalbert:deadline` | Adalbert |
| Contract link | `adalbert:subscribesTo` | Adalbert |

---

## 10. DCON -> Adalbert Migration

Complete mapping of every DCON term to its Adalbert equivalent.

### 10.1 Classes

| DCON Class | Adalbert Equivalent | Status | Notes |
|------------|---------------------|--------|-------|
| `dcon:DataContract` | `adalbert:DataContract` | Direct | Subclass of `odrl:Offer` |
| `dcon:DataContractSubscription` | `adalbert:Subscription` | Direct | Subclass of `odrl:Agreement` |
| `dcon:Promise` | `odrl:Duty` | Dissolved | All obligations are duties |
| `dcon:ProviderPromise` | `odrl:Duty` + DUE action | Dissolved | Action differentiates duty type |
| `dcon:ProviderTimelinessPromise` | Duty + `deliver` + `recurrence` + `deadline` | Pattern | Schedule + window replaces delivery time |
| `dcon:ProviderSchemaPromise` | Duty + `conformTo` | Pattern | Schema conformance duty |
| `dcon:ProviderChangeNotificationPromise` | Duty + `notify` + `deadline` | Pattern | Deadline as duration for lead time |
| `dcon:ProviderQualityPromise` | Duty + `conformTo` + constraint | Pattern | Quality constraints on conformance duty |
| `dcon:ConsumerPromise` | `odrl:Duty` (assignee duty) | Dissolved | Standard ODRL duty |
| `dcon:Schedule` | `adalbert:recurrence` value | Simplified | Single RRULE string, not a class |
| `dcon:ICalSchedule` | `adalbert:recurrence` value | Absorbed | RRULE is the iCal format |
| `dcon:PromiseState` | `adalbert:State` | Extended | 4 states (adds Pending) vs DCON's 3 |
| `dcon:Condition` | `odrl:Constraint` | Standard | Use ODRL directly |
| `dcon:ContractStatus` (Draft/Active/Retired/Cancelled) | -- | Not modeled | Pre-normative workflow metadata |

### 10.2 Properties

| DCON Property | Adalbert Equivalent | Status | Notes |
|---------------|---------------------|--------|-------|
| `dcon:provider` | `odrl:assigner` | Standard | ODRL term |
| `dcon:subscriber` | `odrl:assignee` | Standard | ODRL term |
| `dcon:contractState` | `adalbert:state` | Direct | Unified 4-state lifecycle |
| `dcon:effectiveDate` | `adalbert:effectiveDate` | Direct | Same semantics |
| `dcon:expirationDate` | `adalbert:expirationDate` | Direct | Same semantics |
| `dcon:subscribesTo` | `adalbert:subscribesTo` | Direct | Same semantics |
| `dcon:promisedDeliveryTime` | `adalbert:recurrence` + `adalbert:deadline` | Simplified | Schedule + window |
| `dcon:notificationLeadTime` | `adalbert:deadline` (as `xsd:duration`) | Simplified | Duration value |
| `dcon:promisor` | `adalbert:subject` (on Duty) | Adalbert | `rdfs:subPropertyOf odrl:assignee` |
| `dcon:promisee` | Implicit (other party) | Not needed | Bilateral agreement handles this |
| `dcon:promiseContent` | `odrl:action` (on Duty) | Standard | ODRL term |
| `dcon:promiseState` | `adalbert:state` | Direct | Unified lifecycle |
| `dcon:fulfillsPromise` | `prov:wasDerivedFrom` (on Subscription) | Standard | W3C PROV |
| `dcon:hasPromise` | `odrl:obligation` | Standard | ODRL term |
| `dcon:hasContract` | `dcat:hasPolicy` (external systems) | External | Not policy semantics |
| `dcon:hasEffectivePeriod` | `adalbert:effectiveDate` + `adalbert:expirationDate` | Simplified | Two dates instead of period object |
| `dcon:hasTerminationClause` | `adalbert:deadline` (as duration) | Simplified | Duration on duty |
| `dcon:hasSchedule` | `adalbert:recurrence` | Simplified | Single property |
| `dcon:icalRule` | `adalbert:recurrence` value | Absorbed | RRULE string is the value |

### 10.3 Actions

| DCON Action | Adalbert Equivalent | Status |
|-------------|---------------------|--------|
| `dcon:deliver` | `adalbert-due:deliver` | Direct |
| `dcon:maintain` (schema) | `adalbert-due:conformTo` | Renamed |
| `dcon:notify` | `adalbert-due:notify` | Direct |
| `dcon:report` (implied) | `adalbert-due:report` | Direct |

### 10.4 Not Modeled (by design)

These DCON concepts are intentionally not modeled in Adalbert, with rationale:

| DCON Concept | Reason Not Modeled |
|--------------|--------------------|
| `dcon:Draft` / `dcon:Retired` / `dcon:Cancelled` | Pre-normative workflow metadata, not policy semantics. Belongs to authoring system. |
| `dcon:promiseContent` (as class) | Dissolved into `odrl:action` on duties. Actions provide the semantic differentiation. |
| `dcon:fulfillsPromise` (as property) | Replaced by `prov:wasDerivedFrom` on Subscription. Standard W3C provenance. |
| `dcon:hasContract` | External relationship. Systems use `dcat:hasPolicy` to link datasets to policies. |
| `dcon:Condition` (as class) | Use `odrl:Constraint` directly. Standard ODRL. |
| `schema:Schedule` format | Adalbert uses RRULE string only. Single property instead of class hierarchy. |
| Promise/ProviderPromise hierarchy | Dissolved into `odrl:Duty` patterns. DUE actions differentiate duty types. |
| Promise Theory semantics (ought-to-be) | Duties suffice for Adalbert's scope. RL2 has full Promise support for voluntary cooperation. |
| `dcat:Distribution` integration | Use DCAT directly outside policies. Not policy semantics. |
| Subscription billing | Operational concern, out of scope for policy evaluation. |

### 10.5 Migration Checklist

When migrating a DCON contract to Adalbert v0.7:

1. Change `dcon:DataContract` -> `adalbert:DataContract`
2. Change `dcon:DataContractSubscription` -> `adalbert:Subscription`
3. Add `odrl:profile <https://vocabulary.bigbank/adalbert/>` declaration
4. Replace promise subclasses with `odrl:Duty` + DUE action (see table above)
5. Replace `dcon:promisor` with `adalbert:subject` on duties
6. Replace `dcon:hasSchedule`/`dcon:icalRule` with `adalbert:recurrence`
7. Map `dcon:contractState` to `adalbert:state`
8. Validate against SHACL shapes: `shacl validate --shapes adalbert-shacl.ttl --data migrated.ttl`

See [comparisons/comparison-dcon.md](comparisons/comparison-dcon.md) for SPARQL migration queries and detailed examples.

---

**Version**: 0.7 | **Date**: 2026-02-04
