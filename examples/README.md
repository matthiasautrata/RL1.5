# Adalbert Examples

Concrete policy examples demonstrating Adalbert features.

## Examples

| File | Description |
|------|-------------|
| [market-data-contract.ttl](market-data-contract.ttl) | DataContract, Subscription, bilateral duties, provider SLA |
| [data-use-policy.ttl](data-use-policy.ttl) | Set policy, role-based access, purpose constraints, prohibitions |

## Key Patterns Demonstrated

### 1. Bilateral Agreements (market-data-contract.ttl)

```turtle
ex:subscription a adalbert-dc:Subscription ;
    adalbert:grantor ex:dataTeam ;      # Has duties too!
    adalbert:grantee ex:consumer ;
    
    # Grantor duty (provider SLA)
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:dataTeam ;  # ← Grantor
        adalbert:action adalbert-md:deliver ;
        ...
    ] ;
    
    # Grantee duty (consumer obligation)
    adalbert:clause [
        a adalbert:Duty ;
        adalbert:subject ex:consumer ;  # ← Grantee
        adalbert:action adalbert-md:report ;
        ...
    ] .
```

### 2. Duty Lifecycle

```turtle
ex:duty a adalbert:Duty ;
    adalbert:deadline "P30D"^^xsd:duration ;  # Relative to activation
    adalbert:dutyState adalbert:Pending .     # → Active → Fulfilled/Violated
```

### 3. Hierarchical Matching

```turtle
# Agent hierarchy
ex:analyst adalbert:memberOf ex:team .
ex:team adalbert:memberOf ex:division .

# Asset hierarchy  
ex:table adalbert:partOf ex:schema .
ex:schema adalbert:partOf ex:database .

# Action hierarchy
adalbert-md:display adalbert:includedIn adalbert-gov:use .
```

Policy on `ex:division` applies to `ex:analyst` via transitivity.

### 4. Logical Conditions

```turtle
adalbert:condition [
    a adalbert:LogicalConstraint ;
    adalbert:constraintOperator adalbert:and ;
    adalbert:operand [
        a adalbert:AtomicConstraint ;
        adalbert:leftOperand adalbert-gov:purpose ;
        adalbert:constraintOperator adalbert:eq ;
        adalbert:rightOperand adalbert-gov:analytics
    ] ;
    adalbert:operand [
        a adalbert:AtomicConstraint ;
        adalbert:leftOperand adalbert-du:environment ;
        adalbert:constraintOperator adalbert:eq ;
        adalbert:rightOperand adalbert-du:production
    ]
] .
```

### 5. Contract Versioning

```turtle
ex:contract-v2 a adalbert-dc:DataContract ;
    prov:wasRevisionOf ex:contract-v1 .
```

## Validation

```bash
# Validate examples against SHACL shapes
shacl validate \
  --shapes ../ontology/adalbert-shacl.ttl \
  --data market-data-contract.ttl

shacl validate \
  --shapes ../ontology/adalbert-shacl.ttl \
  --data data-use-policy.ttl
```
