# Adalbert vs ODRL 2.2: Detailed Comparison

**Purpose**: Define precisely how Adalbert relates to, extends, and constrains ODRL 2.2.

---

## Executive Summary

Adalbert is a **semantic clarification and operational tightening** of ODRL 2.2. All Adalbert policies use ODRL vocabulary but impose stricter semantics to eliminate ambiguity and enable formal verification.

| Dimension | ODRL 2.2 | Adalbert |
|-----------|----------|-------|
| Norm types | 3 (implicit Hohfeld) | 3 (explicit semantics) |
| Duty lifecycle | Unspecified | Explicit 3-state machine |
| Pre-conditions | Conflated with duties | Separated |
| Undefined states | Possible | Impossible (totality) |
| Formal semantics | Prose description | Operational semantics |
| Verification target | None | Dafny/Why3 |

---

## Normative Mappings

### Rule Types

| ODRL 2.2 | Adalbert | Notes |
|----------|-------|-------|
| `odrl:permission` | `adalbert:Privilege` | Same intent; Adalbert uses Hohfeldian term |
| `odrl:duty` | `adalbert:Duty` | Adalbert adds explicit lifecycle |
| `odrl:prohibition` | `adalbert:Prohibition` | Identical semantics |
| `odrl:obligation` | — | Adalbert uses `adalbert:Duty` on Agreements |

### Policy Types

| ODRL 2.2 | Adalbert | Notes |
|----------|-------|-------|
| `odrl:Set` | `adalbert:Set` | Identical structure |
| `odrl:Offer` | — | Deferred to DCON profile |
| `odrl:Agreement` | `adalbert:Agreement` | Adds `grantor`/`grantee` requirement |

---

## Semantic Clarifications

### 1. Duty Lifecycle

**ODRL 2.2 Problem**: The specification describes duties but never defines when they are "active" or "violated". Implementations must guess.

```
ODRL 2.2 (vague):
"If the Duty is not satisfied, then that Rule may not be exercised."
```

**Adalbert Solution**: Explicit state machine.

```
DutyState ::= Pending | Active | Fulfilled | Violated

Transitions:
  Pending --[condition becomes true]--> Active
  Active  --[action performed]-------> Fulfilled
  Active  --[deadline passed]--------> Violated
```

### 2. Pre-conditions vs Activation Conditions

**ODRL 2.2 Problem**: `odrl:constraint` on a `duty` is ambiguous:
- Does it mean "this duty only applies if..."?
- Or "the duty holder must ensure that..."?

**Adalbert Solution**: Explicit separation.

```turtle
# Adalbert: Condition is pre-requisite, not obligation
ex:myDuty a adalbert:Duty ;
    adalbert:condition [ ... ] ;   # When this becomes true, duty activates
    adalbert:action ex:notify ;    # What must be done
    adalbert:deadline "P30D"^^xsd:duration .
```

### 3. Constraint Evaluation Order

**ODRL 2.2 Problem**: When multiple constraints exist, evaluation order is undefined.

**Adalbert Solution**: Deterministic semantics.

```
And(c1, c2, ..., cn) evaluates left-to-right, short-circuits on false
Or(c1, c2, ..., cn) evaluates left-to-right, short-circuits on true
Not(c) inverts boolean result
```

### 4. Conflict Resolution

**ODRL 2.2 Problem**: `odrl:conflict` property allows multiple strategies but semantics are vague.

**Adalbert Solution**: Fixed strategy.

```
Prohibition > Privilege
```

If a request matches both a prohibition and a privilege for the same action/asset, denial wins. RL2 offers configurable strategies; Adalbert fixes this for determinism.

---

## Properties: ODRL 2.2 → Adalbert Mapping

### Core Properties (Preserved)

| ODRL 2.2 | Adalbert | Status |
|----------|-------|--------|
| `odrl:action` | `adalbert:action` | Required |
| `odrl:target` | `adalbert:object` | Renamed for clarity |
| `odrl:constraint` | `adalbert:condition` | Semantics clarified |
| `odrl:assignee` | `adalbert:subject` | Generalized |
| `odrl:assigner` | — | Policy-level only |
| `odrl:leftOperand` | `adalbert:leftOperand` | Unchanged |
| `odrl:operator` | `adalbert:constraintOperator` | Unchanged |
| `odrl:rightOperand` | `adalbert:rightOperand` | Unchanged |

### Properties Added by Adalbert

| Property | Purpose |
|----------|---------|
| `adalbert:deadline` | Duty completion time bound |
| `adalbert:dutyState` | Current lifecycle state |
| `adalbert:grantor` | Agreement assigner (required) |
| `adalbert:grantee` | Agreement assignee (required) |

### Properties Restricted in Adalbert

| ODRL 2.2 | Adalbert Status | Reason |
|----------|--------------|--------|
| `odrl:conflict` | Fixed to `prohibit` | Determinism |
| `odrl:inheritFrom` | Not supported | Complexity |
| `odrl:profile` | Required | Must declare `adalbert:` |
| `odrl:remedy` | Not supported | Deferred to RL2 |
| `odrl:consequence` | Not supported | Deferred to RL2 |

---

## Operators

### Comparison Operators (All Supported)

| Operator | ODRL 2.2 | Adalbert |
|----------|----------|-------|
| `odrl:eq` | ✓ | `adalbert:eq` |
| `odrl:neq` | ✓ | `adalbert:neq` |
| `odrl:lt` | ✓ | `adalbert:lt` |
| `odrl:lteq` | ✓ | `adalbert:lte` |
| `odrl:gt` | ✓ | `adalbert:gt` |
| `odrl:gteq` | ✓ | `adalbert:gte` |
| `odrl:isA` | ✓ | Not supported (use hierarchy) |
| `odrl:hasPart` | ✓ | Not supported |
| `odrl:isPartOf` | ✓ | Via `skos:broader` |
| `odrl:isAllOf` | ✓ | `adalbert:isAnyOf` with And |
| `odrl:isAnyOf` | ✓ | `adalbert:isAnyOf` |
| `odrl:isNoneOf` | ✓ | `adalbert:isNoneOf` |

### Logical Operators

| Operator | ODRL 2.2 | Adalbert | Notes |
|----------|----------|-------|-------|
| `odrl:and` | ✓ | `adalbert:and` | |
| `odrl:or` | ✓ | `adalbert:or` | |
| `odrl:xone` | ✓ | — | Deferred to RL2 |
| — | — | `adalbert:not` | Added |

---

## What Adalbert Removes

### Constructs Not in Adalbert

| ODRL 2.2 Construct | Reason for Exclusion |
|--------------------|---------------------|
| `odrl:Offer` | Use `adalbert:Set` (DCON handles offers) |
| `odrl:Ticket` | Domain-specific |
| `odrl:Request` | Separate from policy model |
| `odrl:AssetCollection` | Use explicit listing |
| `odrl:PartyCollection` | Use explicit listing |
| `odrl:inheritAllowed` | Complexity |
| `odrl:remedy` | Deferred to RL2 promises |
| `odrl:consequence` | Deferred to RL2 promises |

### Why These Are Deferred

**Remedy/Consequence**: These imply automatic policy generation (violated duty → new duty appears). RL2 handles this through Promise Theory; Adalbert avoids the complexity.

**AssetCollection/PartyCollection**: Dynamic collection resolution introduces non-determinism. Adalbert requires explicit enumeration; profiles may define resolution mechanisms.

---

## Validation: ODRL 2.2 → Adalbert

A policy conforms to Adalbert if:

1. **Syntax**: Validates against `adalbert-shacl.ttl`
2. **Profile Declaration**: Contains `odrl:profile <https://rl2.org/adalbert/>`
3. **No Excluded Constructs**: Does not use remedy, consequence, xone, collections
4. **Agreement Requirements**: Agreements have exactly one grantor and grantee
5. **Duty Requirements**: All duties have explicit lifecycle tracking

### SHACL Enforcement

```turtle
# Reject RL2-only constructs
rl15-sh:RejectClaimShape a sh:NodeShape ;
    sh:targetClass rl2:Claim ;
    sh:severity sh:Violation ;
    sh:message "Claims are RL2-only; not permitted in Adalbert" .

# Require duty state tracking
rl15-sh:DutyStateShape a sh:NodeShape ;
    sh:targetClass adalbert:Duty ;
    sh:property [
        sh:path adalbert:dutyState ;
        sh:minCount 0 ;  # State may be computed, not stored
    ] .
```

---

## Migration Path

### ODRL 2.2 → Adalbert

1. Add profile declaration
2. Rename `odrl:target` → `adalbert:object` (optional but recommended)
3. Add explicit duty deadlines where implicit
4. Remove remedy/consequence (handle externally)
5. Validate against SHACL shapes

### Adalbert → RL2

1. No policy changes required
2. Add new constructs (Claims, Powers, Promises) as needed
3. Configure conflict resolution strategy if non-default desired

---

## Example: Same Policy in Both

### ODRL 2.2

```turtle
ex:policy a odrl:Set ;
    odrl:permission [
        odrl:target ex:dataset ;
        odrl:action odrl:read ;
        odrl:assignee ex:consumer ;
        odrl:constraint [
            odrl:leftOperand odrl:purpose ;
            odrl:operator odrl:eq ;
            odrl:rightOperand ex:analytics
        ]
    ] .
```

### Adalbert

```turtle
ex:policy a adalbert:Set ;
    odrl:profile <https://rl2.org/adalbert/> ;
    adalbert:clause [
        a adalbert:Privilege ;
        adalbert:subject ex:consumer ;
        adalbert:action odrl:read ;
        adalbert:object ex:dataset ;
        adalbert:condition [
            a adalbert:AtomicConstraint ;
            adalbert:leftOperand odrl:purpose ;
            adalbert:constraintOperator adalbert:eq ;
            adalbert:rightOperand ex:analytics
        ]
    ] .
```

---

## References

- [ODRL Information Model 2.2](https://www.w3.org/TR/odrl-model/)
- [ODRL Vocabulary 2.2](https://www.w3.org/TR/odrl-vocab/)
- [Adalbert Formal Semantics](../RL1_5_Semantics.md)
- [RL2 Full Specification](../../RL2/README.md)
