# Adalbert vs ODRL 2.2: Detailed Comparison

**Purpose**: Define precisely how Adalbert relates to and extends ODRL 2.2.

---

## Executive Summary

Adalbert is a **proper ODRL 2.2 profile**. It uses ODRL terms directly for all standard constructs and adds a small set of extensions for deterministic evaluation. Every Adalbert policy is a valid ODRL 2.2 policy.

| Dimension | ODRL 2.2 | Adalbert Profile |
|-----------|----------|------------------|
| Vocabulary | ODRL terms | Same ODRL terms |
| Duty lifecycle | Unspecified | Explicit 4-state machine (extension) |
| Pre-conditions | Conflated with duties | Separated via lifecycle semantics |
| Undefined states | Possible | Impossible (totality) |
| Formal semantics | Prose description | Operational semantics |
| Verification target | None | Dafny/Why3 |
| Conflict resolution | Configurable | Fixed: Prohibition > Permission |

---

## What Adalbert Uses from ODRL (Directly)

Adalbert uses the following ODRL terms without renaming, sub-classing, or wrapping:

### Rule Types

| ODRL Term | Used Directly |
|-----------|--------------|
| `odrl:Permission` | Yes |
| `odrl:Duty` | Yes (Adalbert adds `deadline` and `state` properties) |
| `odrl:Prohibition` | Yes |
| `odrl:obligation` | Yes (duties on Agreements) |

### Policy Types

| ODRL Term | Used Directly | Notes |
|-----------|--------------|-------|
| `odrl:Set` | Yes | |
| `odrl:Offer` | Yes (via `adalbert:DataContract` subclass) | |
| `odrl:Agreement` | Yes (via `adalbert:Subscription` subclass) | |

### Core Properties

| ODRL Property | Used Directly |
|---------------|--------------|
| `odrl:permission` | Yes |
| `odrl:prohibition` | Yes |
| `odrl:obligation` | Yes |
| `odrl:action` | Yes |
| `odrl:target` | Yes |
| `odrl:constraint` | Yes |
| `odrl:assignee` | Yes |
| `odrl:assigner` | Yes |
| `odrl:leftOperand` | Yes |
| `odrl:operator` | Yes |
| `odrl:rightOperand` | Yes |
| `odrl:includedIn` | Yes (action hierarchy) |

### Constraint Types

| ODRL Term | Used Directly |
|-----------|--------------|
| `odrl:Constraint` | Yes |
| `odrl:LogicalConstraint` | Yes |

### Operators

| ODRL Operator | Used Directly |
|---------------|--------------|
| `odrl:eq` | Yes |
| `odrl:neq` | Yes |
| `odrl:lt` | Yes |
| `odrl:lteq` | Yes |
| `odrl:gt` | Yes |
| `odrl:gteq` | Yes |
| `odrl:isAnyOf` | Yes |
| `odrl:isNoneOf` | Yes |
| `odrl:and` | Yes (on LogicalConstraint) |
| `odrl:or` | Yes (on LogicalConstraint) |

Adalbert adds one operator: `adalbert:not` (logical negation on `odrl:LogicalConstraint`). ODRL defines `odrl:and` and `odrl:or` but lacks negation.

---

## What Adalbert Adds to ODRL

These are the only terms in the `adalbert:` namespace:

### Lifecycle State

| Adalbert Term | Purpose |
|---------------|---------|
| `adalbert:State` | Enumerated lifecycle state class |
| `adalbert:Pending` | Condition not yet met / not yet in force |
| `adalbert:Active` | Condition met, action required / in force |
| `adalbert:Fulfilled` | Action performed / obligations complete |
| `adalbert:Violated` | Deadline passed / breached |
| `adalbert:state` | Property linking duties and contracts to their current state |
| `adalbert:deadline` | Time constraint for duty fulfillment (xsd:dateTime or xsd:duration) |

### Data Contract Classes

| Adalbert Term | Purpose |
|---------------|---------|
| `adalbert:DataContract` | `rdfs:subClassOf odrl:Offer` — data provider's terms |
| `adalbert:Subscription` | `rdfs:subClassOf odrl:Agreement` — activated contract |
| `adalbert:subscribesTo` | Links Subscription to its DataContract |
| `adalbert:effectiveDate` | When contract/subscription becomes effective |
| `adalbert:expirationDate` | When contract/subscription expires |

### Hierarchy Extensions

| Adalbert Term | Purpose |
|---------------|---------|
| `adalbert:partOf` | Transitive asset containment (on `odrl:Asset`) |
| `adalbert:memberOf` | Transitive party membership (on `odrl:Party`) |

### Operand Resolution

| Adalbert Term | Purpose |
|---------------|---------|
| `adalbert:resolutionPath` | Dot-separated path from canonical root to value (on `odrl:LeftOperand`) |

### Runtime References

| Adalbert Term | Purpose |
|---------------|---------|
| `adalbert:RuntimeReference` | Value resolved at evaluation time |
| `adalbert:currentAgent` | The requesting agent |
| `adalbert:currentDateTime` | Evaluation timestamp (`skos:exactMatch odrl:dateTime`) |

### Logical Negation

| Adalbert Term | Purpose |
|---------------|---------|
| `adalbert:not` | Logical negation on `odrl:LogicalConstraint` |

---

## Semantic Clarifications

Adalbert does not rename ODRL terms but imposes stricter semantics:

### 1. Duty Lifecycle

**ODRL 2.2 Problem**: The specification describes duties but never defines when they are "active" or "violated". Implementations must guess.

**Adalbert Solution**: Explicit state machine via `adalbert:State` and `adalbert:state`.

```
State ::= Pending | Active | Fulfilled | Violated

Transitions:
  Pending --[condition becomes true]--> Active
  Active  --[action performed]-------> Fulfilled
  Active  --[deadline passed]--------> Violated
```

### 2. Pre-conditions vs Activation Conditions

**ODRL 2.2 Problem**: `odrl:constraint` on a duty is ambiguous:
- Does it mean "this duty only applies if..."?
- Or "the duty holder must ensure that..."?

**Adalbert Solution**: `odrl:constraint` always means a pre-condition. The lifecycle state machine separates "when does this activate" (Pending to Active transition) from "what must be done" (`odrl:action`).

```turtle
# Constraint is a pre-condition that gates activation
ex:myDuty a odrl:Duty ;
    odrl:constraint [
        a odrl:Constraint ;
        odrl:leftOperand adalbert:currentDateTime ;
        odrl:operator odrl:gteq ;
        odrl:rightOperand "06:00:00"^^xsd:time
    ] ;
    odrl:action ex:notify ;
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
Prohibition > Permission
```

If a request matches both a prohibition and a permission for the same action/asset, denial wins. RL2 offers configurable strategies; Adalbert fixes this for determinism.

---

## What Adalbert Restricts

### Constructs Not Permitted in Adalbert Policies

| ODRL 2.2 Construct | Reason for Exclusion |
|--------------------|---------------------|
| `odrl:Ticket` | Domain-specific |
| `odrl:Request` | Separate from policy model |
| `odrl:AssetCollection` | Use explicit listing |
| `odrl:PartyCollection` | Use explicit listing |
| `odrl:inheritAllowed` | Complexity |
| `odrl:inheritFrom` | Complexity |
| `odrl:remedy` | Deferred to RL2 promises |
| `odrl:consequence` | Deferred to RL2 promises |
| `odrl:xone` | Deferred to RL2 |

### Properties Fixed or Restricted

| ODRL 2.2 | Adalbert Status | Reason |
|----------|--------------|--------|
| `odrl:conflict` | Fixed to `odrl:prohibit` | Determinism |
| `odrl:profile` | Required | Must declare `<https://vocabulary.bigbank/adalbert/>` |

### Why These Are Deferred

**Remedy/Consequence**: These imply automatic policy generation (violated duty leads to new duty). RL2 handles this through Promise Theory; Adalbert avoids the complexity.

**AssetCollection/PartyCollection**: Dynamic collection resolution introduces non-determinism. Adalbert requires explicit enumeration; profiles may define resolution mechanisms.

---

## Validation: ODRL 2.2 to Adalbert

A policy conforms to Adalbert if:

1. **Syntax**: Validates against `adalbert-shacl.ttl`
2. **Profile Declaration**: Contains `odrl:profile <https://vocabulary.bigbank/adalbert/>`
3. **No Excluded Constructs**: Does not use remedy, consequence, xone, collections
4. **Agreement Requirements**: Agreements have `odrl:assigner` and `odrl:assignee`
5. **Duty Requirements**: All duties support explicit lifecycle tracking

### SHACL Enforcement

```turtle
# Reject RL2-only constructs
adalbertsh:RejectClaimShape a sh:NodeShape ;
    sh:targetClass rl2:Claim ;
    sh:severity sh:Violation ;
    sh:message "Claims are RL2-only; not permitted in Adalbert" .

# Duty shape — allow state tracking
adalbertsh:DutyShape a sh:NodeShape ;
    sh:targetClass odrl:Duty ;
    sh:property [
        sh:path adalbert:state ;
        sh:minCount 0 ;  # State may be computed, not stored
    ] .
```

---

## Migration Path

### ODRL 2.2 to Adalbert

1. Add profile declaration: `odrl:profile <https://vocabulary.bigbank/adalbert/>`
2. Add explicit duty deadlines where currently implicit
3. Remove remedy/consequence (handle externally)
4. Validate against SHACL shapes

No term renaming is required. Adalbert uses ODRL terms directly.

### Adalbert to RL2

1. No policy changes required
2. Add new constructs (Claims, Powers, Promises) as needed
3. Configure conflict resolution strategy if non-default desired

---

## Example: Adalbert Policy IS a Valid ODRL Policy

An Adalbert policy uses ODRL terms throughout. The `adalbert:` extensions (deadline, state) are additive and ignored by standard ODRL processors.

### Adalbert Policy

```turtle
@prefix odrl:         <http://www.w3.org/ns/odrl/2/> .
@prefix adalbert:     <https://vocabulary.bigbank/adalbert/> .
@prefix adalbert-due: <https://vocabulary.bigbank/adalbert/due/> .

ex:policy a odrl:Set ;
    odrl:profile <https://vocabulary.bigbank/adalbert/> ;

    odrl:permission [
        a odrl:Permission ;
        odrl:assignee ex:consumer ;
        odrl:action odrl:read ;
        odrl:target ex:dataset ;
        odrl:constraint [
            a odrl:Constraint ;
            odrl:leftOperand odrl:purpose ;
            odrl:operator odrl:eq ;
            odrl:rightOperand ex:analytics
        ]
    ] ;

    odrl:obligation [
        a odrl:Duty ;
        odrl:assignee ex:consumer ;
        odrl:action adalbert-due:log ;
        odrl:target ex:accessLog ;
        adalbert:deadline "PT1H"^^xsd:duration ;    # Adalbert extension
        adalbert:state adalbert:Pending              # Adalbert extension
    ] .
```

A standard ODRL 2.2 processor sees valid `odrl:Set`, `odrl:Permission`, `odrl:Duty` etc. It ignores `adalbert:deadline` and `adalbert:state` as unknown properties. An Adalbert-aware processor uses the extensions for lifecycle tracking and deadline enforcement.

---

## References

- [ODRL Information Model 2.2](https://www.w3.org/TR/odrl-model/)
- [ODRL Vocabulary 2.2](https://www.w3.org/TR/odrl-vocab/)
- [Adalbert Formal Semantics](../Adalbert_Semantics.md)
- [RL2 Full Specification](../../RL2/README.md)
