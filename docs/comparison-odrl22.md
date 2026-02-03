# RL1.5 vs ODRL 2.2: Detailed Comparison

**Purpose**: Define precisely how RL1.5 relates to, extends, and constrains ODRL 2.2.

---

## Executive Summary

RL1.5 is a **semantic clarification and operational tightening** of ODRL 2.2. All RL1.5 policies use ODRL vocabulary but impose stricter semantics to eliminate ambiguity and enable formal verification.

| Dimension | ODRL 2.2 | RL1.5 |
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

| ODRL 2.2 | RL1.5 | Notes |
|----------|-------|-------|
| `odrl:permission` | `rl15:Privilege` | Same intent; RL1.5 uses Hohfeldian term |
| `odrl:duty` | `rl15:Duty` | RL1.5 adds explicit lifecycle |
| `odrl:prohibition` | `rl15:Prohibition` | Identical semantics |
| `odrl:obligation` | — | RL1.5 uses `rl15:Duty` on Agreements |

### Policy Types

| ODRL 2.2 | RL1.5 | Notes |
|----------|-------|-------|
| `odrl:Set` | `rl15:Set` | Identical structure |
| `odrl:Offer` | — | Deferred to DCON profile |
| `odrl:Agreement` | `rl15:Agreement` | Adds `grantor`/`grantee` requirement |

---

## Semantic Clarifications

### 1. Duty Lifecycle

**ODRL 2.2 Problem**: The specification describes duties but never defines when they are "active" or "violated". Implementations must guess.

```
ODRL 2.2 (vague):
"If the Duty is not satisfied, then that Rule may not be exercised."
```

**RL1.5 Solution**: Explicit state machine.

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

**RL1.5 Solution**: Explicit separation.

```turtle
# RL1.5: Condition is pre-requisite, not obligation
ex:myDuty a rl15:Duty ;
    rl15:condition [ ... ] ;   # When this becomes true, duty activates
    rl15:action ex:notify ;    # What must be done
    rl15:deadline "P30D"^^xsd:duration .
```

### 3. Constraint Evaluation Order

**ODRL 2.2 Problem**: When multiple constraints exist, evaluation order is undefined.

**RL1.5 Solution**: Deterministic semantics.

```
And(c1, c2, ..., cn) evaluates left-to-right, short-circuits on false
Or(c1, c2, ..., cn) evaluates left-to-right, short-circuits on true
Not(c) inverts boolean result
```

### 4. Conflict Resolution

**ODRL 2.2 Problem**: `odrl:conflict` property allows multiple strategies but semantics are vague.

**RL1.5 Solution**: Fixed strategy.

```
Prohibition > Privilege
```

If a request matches both a prohibition and a privilege for the same action/asset, denial wins. RL2 offers configurable strategies; RL1.5 fixes this for determinism.

---

## Properties: ODRL 2.2 → RL1.5 Mapping

### Core Properties (Preserved)

| ODRL 2.2 | RL1.5 | Status |
|----------|-------|--------|
| `odrl:action` | `rl15:action` | Required |
| `odrl:target` | `rl15:object` | Renamed for clarity |
| `odrl:constraint` | `rl15:condition` | Semantics clarified |
| `odrl:assignee` | `rl15:subject` | Generalized |
| `odrl:assigner` | — | Policy-level only |
| `odrl:leftOperand` | `rl15:leftOperand` | Unchanged |
| `odrl:operator` | `rl15:constraintOperator` | Unchanged |
| `odrl:rightOperand` | `rl15:rightOperand` | Unchanged |

### Properties Added by RL1.5

| Property | Purpose |
|----------|---------|
| `rl15:deadline` | Duty completion time bound |
| `rl15:dutyState` | Current lifecycle state |
| `rl15:grantor` | Agreement assigner (required) |
| `rl15:grantee` | Agreement assignee (required) |

### Properties Restricted in RL1.5

| ODRL 2.2 | RL1.5 Status | Reason |
|----------|--------------|--------|
| `odrl:conflict` | Fixed to `prohibit` | Determinism |
| `odrl:inheritFrom` | Not supported | Complexity |
| `odrl:profile` | Required | Must declare `rl15:` |
| `odrl:remedy` | Not supported | Deferred to RL2 |
| `odrl:consequence` | Not supported | Deferred to RL2 |

---

## Operators

### Comparison Operators (All Supported)

| Operator | ODRL 2.2 | RL1.5 |
|----------|----------|-------|
| `odrl:eq` | ✓ | `rl15:eq` |
| `odrl:neq` | ✓ | `rl15:neq` |
| `odrl:lt` | ✓ | `rl15:lt` |
| `odrl:lteq` | ✓ | `rl15:lte` |
| `odrl:gt` | ✓ | `rl15:gt` |
| `odrl:gteq` | ✓ | `rl15:gte` |
| `odrl:isA` | ✓ | Not supported (use hierarchy) |
| `odrl:hasPart` | ✓ | Not supported |
| `odrl:isPartOf` | ✓ | Via `skos:broader` |
| `odrl:isAllOf` | ✓ | `rl15:isAnyOf` with And |
| `odrl:isAnyOf` | ✓ | `rl15:isAnyOf` |
| `odrl:isNoneOf` | ✓ | `rl15:isNoneOf` |

### Logical Operators

| Operator | ODRL 2.2 | RL1.5 | Notes |
|----------|----------|-------|-------|
| `odrl:and` | ✓ | `rl15:and` | |
| `odrl:or` | ✓ | `rl15:or` | |
| `odrl:xone` | ✓ | — | Deferred to RL2 |
| — | — | `rl15:not` | Added |

---

## What RL1.5 Removes

### Constructs Not in RL1.5

| ODRL 2.2 Construct | Reason for Exclusion |
|--------------------|---------------------|
| `odrl:Offer` | Use `rl15:Set` (DCON handles offers) |
| `odrl:Ticket` | Domain-specific |
| `odrl:Request` | Separate from policy model |
| `odrl:AssetCollection` | Use explicit listing |
| `odrl:PartyCollection` | Use explicit listing |
| `odrl:inheritAllowed` | Complexity |
| `odrl:remedy` | Deferred to RL2 promises |
| `odrl:consequence` | Deferred to RL2 promises |

### Why These Are Deferred

**Remedy/Consequence**: These imply automatic policy generation (violated duty → new duty appears). RL2 handles this through Promise Theory; RL1.5 avoids the complexity.

**AssetCollection/PartyCollection**: Dynamic collection resolution introduces non-determinism. RL1.5 requires explicit enumeration; profiles may define resolution mechanisms.

---

## Validation: ODRL 2.2 → RL1.5

A policy conforms to RL1.5 if:

1. **Syntax**: Validates against `rl1_5-shacl.ttl`
2. **Profile Declaration**: Contains `odrl:profile <https://rl2.org/rl1.5/>`
3. **No Excluded Constructs**: Does not use remedy, consequence, xone, collections
4. **Agreement Requirements**: Agreements have exactly one grantor and grantee
5. **Duty Requirements**: All duties have explicit lifecycle tracking

### SHACL Enforcement

```turtle
# Reject RL2-only constructs
rl15-sh:RejectClaimShape a sh:NodeShape ;
    sh:targetClass rl2:Claim ;
    sh:severity sh:Violation ;
    sh:message "Claims are RL2-only; not permitted in RL1.5" .

# Require duty state tracking
rl15-sh:DutyStateShape a sh:NodeShape ;
    sh:targetClass rl15:Duty ;
    sh:property [
        sh:path rl15:dutyState ;
        sh:minCount 0 ;  # State may be computed, not stored
    ] .
```

---

## Migration Path

### ODRL 2.2 → RL1.5

1. Add profile declaration
2. Rename `odrl:target` → `rl15:object` (optional but recommended)
3. Add explicit duty deadlines where implicit
4. Remove remedy/consequence (handle externally)
5. Validate against SHACL shapes

### RL1.5 → RL2

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

### RL1.5

```turtle
ex:policy a rl15:Set ;
    odrl:profile <https://rl2.org/rl1.5/> ;
    rl15:clause [
        a rl15:Privilege ;
        rl15:subject ex:consumer ;
        rl15:action odrl:read ;
        rl15:object ex:dataset ;
        rl15:condition [
            a rl15:AtomicConstraint ;
            rl15:leftOperand odrl:purpose ;
            rl15:constraintOperator rl15:eq ;
            rl15:rightOperand ex:analytics
        ]
    ] .
```

---

## References

- [ODRL Information Model 2.2](https://www.w3.org/TR/odrl-model/)
- [ODRL Vocabulary 2.2](https://www.w3.org/TR/odrl-vocab/)
- [RL1.5 Formal Semantics](../RL1_5_Semantics.md)
- [RL2 Full Specification](../../RL2/README.md)
