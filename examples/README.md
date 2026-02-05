# Adalbert Examples

Concrete policy examples demonstrating Adalbert as an ODRL 2.2 profile.

Every Adalbert policy is a valid ODRL 2.2 policy. The examples below use ODRL terms
directly (`odrl:obligation`, `odrl:constraint`, `odrl:assigner`, `odrl:assignee`, etc.)
and extend them with Adalbert-specific vocabulary only where ODRL has no equivalent
(lifecycle state, deadlines, hierarchical membership).

## Examples

| File | Description |
|------|-------------|
| [data-contract.ttl](data-contract.ttl) | DataContract, Subscription, bilateral duties, provider SLA |
| [data-use-policy.ttl](data-use-policy.ttl) | Set policy, role-based access, purpose constraints, prohibitions |

## Key Patterns Demonstrated

### 1. Bilateral Agreements (data-contract.ttl)

Adalbert models bilateral agreements using ODRL's `odrl:obligation` with
`odrl:assignee` on each duty to distinguish provider duties from consumer duties.

```turtle
ex:subscription a adalbert:Subscription ;
    odrl:assigner ex:dataTeam ;
    odrl:assignee ex:consumer ;
    odrl:obligation [
        a odrl:Duty ;
        odrl:assignee ex:dataTeam ;
        odrl:action adalbert-due:deliver ;
        ...
    ] ;
    odrl:obligation [
        a odrl:Duty ;
        odrl:assignee ex:consumer ;
        odrl:action adalbert-due:report ;
        ...
    ] .
```

### 2. Unified Lifecycle State

Adalbert adds lifecycle state tracking to ODRL duties and contracts.
Four states (Pending, Active, Fulfilled, Violated) apply uniformly.

```turtle
ex:duty a odrl:Duty ;
    adalbert:deadline "P30D"^^xsd:duration ;
    adalbert:state adalbert:Pending .

ex:contract a adalbert:DataContract ;
    adalbert:state adalbert:Active .
```

### 3. Hierarchical Matching

Adalbert extends ODRL with hierarchical membership for agents and assets,
and reuses ODRL's `odrl:includedIn` for action subsumption.

```turtle
ex:analyst adalbert:memberOf ex:team .
ex:table adalbert:partOf ex:schema .
odrl:display odrl:includedIn odrl:use .
```

Policy on `ex:team` applies to `ex:analyst` via transitivity.

### 4. Logical Conditions

Adalbert uses ODRL's `odrl:LogicalConstraint` pattern for compound conditions.

```turtle
odrl:constraint [
    a odrl:LogicalConstraint ;
    odrl:and (
        [
            a odrl:Constraint ;
            odrl:leftOperand odrl:purpose ;
            odrl:operator odrl:eq ;
            odrl:rightOperand adalbert-due:analytics
        ]
        [
            a odrl:Constraint ;
            odrl:leftOperand adalbert-due:environment ;
            odrl:operator odrl:eq ;
            odrl:rightOperand adalbert-due:production
        ]
    )
] .
```

### 5. Recurring Duties

Adalbert supports recurring duties via `adalbert:recurrence` — an RFC 5545 RRULE string that defines when duty instances are generated. The `deadline` defines the per-instance fulfillment window.

```turtle
odrl:obligation [
    a odrl:Duty ;
    odrl:assignee ex:dataTeam ;
    odrl:action adalbert-due:deliver ;
    odrl:target ex:marketPrices ;
    adalbert:recurrence "FREQ=DAILY;BYHOUR=6;BYMINUTE=0" ;
    adalbert:deadline "06:30:00"^^xsd:time
] .
```

This creates a daily delivery duty at 06:00 with a 30-minute fulfillment window. Each instance follows the standard lifecycle independently.

### 6. Contract Versioning

```turtle
ex:contract-v2 a adalbert:DataContract ;
    prov:wasRevisionOf ex:contract-v1 .
```

## Validation

```bash
# Validate examples against SHACL shapes
shacl validate \
  --shapes ../ontology/adalbert-shacl.ttl \
  --data data-contract.ttl

shacl validate \
  --shapes ../ontology/adalbert-shacl.ttl \
  --data data-use-policy.ttl
```
