---
title: "RL1.5 Formal Semantics"
subtitle: "Deterministic Policy Evaluation for Data Governance"
version: "0.2"
status: "Draft"
date: 2026-02-03
abstract: |
  RL1.5 extends ODRL 2.2 with deterministic, total evaluation semantics
  and bilateral agreement support. This document specifies the formal
  semantics in a style amenable to mechanization in Dafny, Why3, or
  similar verification frameworks.
---

## 1. Introduction

RL1.5 addresses semantic gaps in ODRL 2.2:

1. **Duty Ambiguity**: ODRL conflates pre-conditions with post-obligations
2. **Unilateral Agreements**: ODRL evaluates only from assignee perspective
3. **Undefined States**: ODRL permits evaluation to be "undefined"

RL1.5 provides:

- Explicit duty lifecycle with state machine semantics
- Bilateral agreement evaluation (grantor and grantee duties)
- Total evaluation functions (always terminate with defined result)
- Clear separation of Condition (pre-requisite) from Duty (obligation)

### 1.1 Notation

- `×` for Cartesian product
- `→` for function types
- `∪` for union
- `∈` for set membership
- `⊥` for undefined/bottom value
- `⟦e⟧` for denotation of expression `e`

### 1.2 Document Status

This document is **normative** for RL1.5 implementations.

---

## 2. Abstract Syntax

### 2.1 Syntactic Domains

| Domain | Symbol | Description |
|--------|--------|-------------|
| Agents | **A** | Set of agent identifiers |
| Actions | **X** | Set of action identifiers |
| Assets | **S** | Set of asset identifiers |
| Values | **V** | Set of atomic values (strings, numbers, URIs) |
| Time | **T** | Time domain (ISO 8601 instants) |
| Duration | **D** | Duration domain (ISO 8601 durations) |

### 2.2 Norms

```
Norm ::= Privilege(subject: Agent, action: Action, asset: Asset, condition: Condition?)
       | Duty(subject: Agent, action: Action, asset: Asset, condition: Condition?, deadline: Deadline?)
       | Prohibition(subject: Agent, action: Action, asset: Asset, condition: Condition?)

Deadline ::= AbsoluteDeadline(time: Time)
           | RelativeDeadline(duration: Duration)
```

**Notes**:

- `AbsoluteDeadline`: Fixed point in time (e.g., 2026-12-31T23:59:59Z)
- `RelativeDeadline`: Duration from activation (e.g., P30D, PT24H)

### 2.3 Conditions

```
Condition ::= AtomicConstraint(leftOperand: LeftOperand, 
                               operator: ComparisonOperator, 
                               rightOperand: Value | RuntimeRef)
            | And(operands: Condition+)
            | Or(operands: Condition+)
            | Not(operand: Condition)

ComparisonOperator ::= eq | neq | lt | lte | gt | gte | isAnyOf | isNoneOf

RuntimeRef ::= currentAgent | currentDateTime
```

### 2.4 Policies

```
Policy ::= Set(target: Asset?, clauses: Norm+, condition: Condition?)
         | Offer(grantor: Agent, grantee: Agent?, target: Asset?, clauses: Norm+, condition: Condition?)
         | Agreement(grantor: Agent, grantee: Agent, target: Asset?, clauses: Norm+, condition: Condition?)
```

**Notes**:

- `Set`: Unilateral declaration, no parties
- `Offer`: Proposal from grantor, grantee optional (open offer)
- `Agreement`: Bilateral binding, both parties identified

### 2.5 Requests

```
Request ::= Request(agent: Agent, action: Action, asset: Asset, context: Context)

Context ::= Map<LeftOperand, Value>
```

---

## 3. Semantic Domains

### 3.1 State

```
Σ = {
    clock        : Time,
    dutyState    : Duty → DutyState,
    activatedAt  : Duty → Time?,           // When duty became Active
    performed    : Set<(Agent, Action, Asset, Time)>
}

DutyState ::= Pending | Active | Fulfilled | Violated
```

**Initial state** Σ₀:

```
Σ₀ = {
    clock       = currentSystemTime,
    dutyState   = λd. Pending,
    activatedAt = λd. ⊥,
    performed   = ∅
}
```

### 3.2 Environment

```
Env = {
    agent   : Agent,
    action  : Action,
    asset   : Asset,
    context : Context,
    state   : Σ
}
```

### 3.3 Decision

```
Decision ::= Permit | Deny | NotApplicable
```

### 3.4 Evaluation Result

```
Result = {
    decision      : Decision,
    grantorDuties : Set<Duty>,    // Duties on the grantor (data provider)
    granteeDuties : Set<Duty>,    // Duties on the grantee (data consumer)
    violations    : Set<Duty>,
    explanation   : Explanation
}
```

---

## 4. Duty Lifecycle

### 4.1 State Diagram

```
                    condition becomes true
         ┌─────────────────────────────────────┐
         │                                     ▼
     ┌───────┐                            ┌────────┐
     │Pending│                            │ Active │
     └───────┘                            └────────┘
                                           │     │
                         action performed  │     │  deadline exceeded
                                           ▼     ▼
                                    ┌──────────┐ ┌─────────┐
                                    │Fulfilled │ │Violated │
                                    └──────────┘ └─────────┘
```

### 4.2 Transition Rules

**Activation** (Pending → Active):

```
duty.condition = ⊥ ∨ ⟦duty.condition⟧(Env) = true
─────────────────────────────────────────────────
Σ.dutyState(duty) := Active
Σ.activatedAt(duty) := Σ.clock
```

**Fulfillment** (Active → Fulfilled):

```
Σ.dutyState(duty) = Active ∧ 
∃(a, x, s, t) ∈ Σ.performed.
    a = duty.subject ∧
    x matches duty.action ∧
    s matches duty.asset ∧
    t ≤ effectiveDeadline(duty, Σ)
────────────────────────────────────────
Σ.dutyState(duty) := Fulfilled
```

**Violation** (Active → Violated):

```
Σ.dutyState(duty) = Active ∧
Σ.clock > effectiveDeadline(duty, Σ) ∧
∄(a, x, s, t) ∈ Σ.performed.
    a = duty.subject ∧ x matches duty.action ∧ s matches duty.asset
────────────────────────────────────────────────────────────────────
Σ.dutyState(duty) := Violated
```

### 4.3 Effective Deadline

```
effectiveDeadline(duty, Σ) =
    case duty.deadline of
        ⊥                        → ∞
        AbsoluteDeadline(t)      → t
        RelativeDeadline(d)      → Σ.activatedAt(duty) + d
```

### 4.4 Match Semantics

Action and asset matching support hierarchy subsumption:

```
x₁ matches x₂ ⟺ x₁ = x₂ ∨ x₁ includedIn⁺ x₂
s₁ matches s₂ ⟺ s₁ = s₂ ∨ s₁ partOf⁺ s₂
```

Where `includedIn⁺` and `partOf⁺` are transitive closures.

---

## 5. Condition Evaluation

### 5.1 Denotational Semantics

```
⟦_⟧ : Condition × Env → Boolean
```

**Atomic constraints**:

```
⟦AtomicConstraint(left, op, right)⟧(Env) =
    let leftVal = resolve(left, Env)
    let rightVal = case right of
        RuntimeRef(r) → resolveRuntime(r, Env)
        Literal(v)    → v
    in if leftVal = ⊥ then false
       else apply(op, leftVal, rightVal)
```

**Logical connectives**:

```
⟦And(c₁, ..., cₙ)⟧(Env) = ⟦c₁⟧(Env) ∧ ... ∧ ⟦cₙ⟧(Env)
⟦Or(c₁, ..., cₙ)⟧(Env)  = ⟦c₁⟧(Env) ∨ ... ∨ ⟦cₙ⟧(Env)
⟦Not(c)⟧(Env)           = ¬⟦c⟧(Env)
```

### 5.2 Resolution Functions

**resolve** : LeftOperand × Env → Value

```
resolve(operand, Env) =
    let key = operand.contextKey
    in if key ∈ Env.context.keys then
           Env.context[key]
       else
           ⊥
```

**resolveRuntime** : RuntimeRef × Env → Value

```
resolveRuntime(ref, Env) =
    case ref of
        currentAgent    → Env.agent
        currentDateTime → Env.state.clock
```

**apply** : ComparisonOperator × Value × Value → Boolean

```
apply(eq, v₁, v₂)       = v₁ = v₂
apply(neq, v₁, v₂)      = v₁ ≠ v₂
apply(lt, v₁, v₂)       = v₁ < v₂
apply(lte, v₁, v₂)      = v₁ ≤ v₂
apply(gt, v₁, v₂)       = v₁ > v₂
apply(gte, v₁, v₂)      = v₁ ≥ v₂
apply(isAnyOf, v, set)  = v ∈ set
apply(isNoneOf, v, set) = v ∉ set
```

---

## 6. Policy Evaluation

### 6.1 Top-Level Evaluation

```
Eval : Request × Set<Policy> × Σ → Result
```

```
Eval(request, policies, Σ) =
    let Env = buildEnv(request, Σ)
    
    // 1. Find applicable policies
    let applicable = { p ∈ policies | PolicyApplicable(p, Env) }
    
    // 2. Collect matching norms
    let prohibitions = collectProhibitions(applicable, Env)
    let privileges = collectPrivileges(applicable, Env)
    let allDuties = collectDuties(applicable, Env)
    
    // 3. Partition duties by bearer (bilateral)
    let grantorDuties = { d ∈ allDuties | d.subject = policy.grantor }
    let granteeDuties = { d ∈ allDuties | d.subject = policy.grantee }
    
    // 4. Update duty states
    let Σ' = updateDutyStates(allDuties, Σ)
    
    // 5. Check for active prohibitions
    if ∃p ∈ prohibitions. NormActive(p, Env) then
        return {decision: Deny, ...}
    
    // 6. Check for granting privilege
    if ∄p ∈ privileges. NormActive(p, Env) then
        return {decision: NotApplicable, ...}
    
    // 7. Check for violations
    let violated = { d ∈ allDuties | Σ'.dutyState(d) = Violated }
    if violated ≠ ∅ then
        return {decision: Deny, violations: violated, ...}
    
    // 8. Collect active duties for both parties
    let activeGrantor = { d ∈ grantorDuties | Σ'.dutyState(d) = Active }
    let activeGrantee = { d ∈ granteeDuties | Σ'.dutyState(d) = Active }
    
    return {
        decision: Permit,
        grantorDuties: activeGrantor,
        granteeDuties: activeGrantee,
        violations: ∅
    }
```

### 6.2 Policy Applicability

```
PolicyApplicable(p, Env) =
    // Target matches (if specified)
    (p.target = ⊥ ∨ Env.asset matches p.target) ∧
    // Policy condition satisfied
    (p.condition = ⊥ ∨ ⟦p.condition⟧(Env))
```

For Agreements, the agent must be a party:

```
PolicyApplicable(Agreement(grantor, grantee, ...), Env) =
    ... ∧ (Env.agent = grantee ∨ Env.agent = grantor)
```

### 6.3 Norm Activation

```
NormActive(norm, Env) =
    (norm.subject = Env.agent ∨ norm.subject = *) ∧
    norm.action matches Env.action ∧
    norm.asset matches Env.asset ∧
    (norm.condition = ⊥ ∨ ⟦norm.condition⟧(Env))
```

---

## 7. Conflict Resolution

### 7.1 Strategy: Prohibition Overrides

RL1.5 uses a fixed conflict resolution:

```
Prohibition > Privilege
```

If both a prohibition and privilege match, the prohibition wins.

### 7.2 Algorithm

```
resolve_conflicts(prohibitions, privileges) =
    if ∃p ∈ prohibitions. active(p) then
        Deny
    else if ∃p ∈ privileges. active(p) then
        Permit
    else
        NotApplicable
```

No specificity ordering within norm types. All matching norms contribute.

---

## 8. Correctness Properties

### 8.1 Totality

**Theorem**: For all well-formed inputs, `Eval` terminates with a defined result.

```
∀ request, policies, Σ.
    WellFormed(request) ∧ WellFormed(policies) ∧ WellFormed(Σ)
    ⟹ ∃ result. Eval(request, policies, Σ) = result ∧ result ≠ ⊥
```

### 8.2 Determinism

**Theorem**: Evaluation is deterministic.

```
∀ request, policies, Σ.
    Eval(request, policies, Σ) = r₁ ∧ Eval(request, policies, Σ) = r₂
    ⟹ r₁ = r₂
```

### 8.3 Prohibition Monotonicity

**Theorem**: Adding policies cannot remove prohibitions.

```
∀ request, P, P', Σ.
    P ⊆ P' ∧ Eval(request, P, Σ).decision = Deny
    ⟹ Eval(request, P', Σ).decision = Deny
```

### 8.4 Duty Lifecycle Invariants

**Theorem**: Terminal states are permanent.

```
∀ d, Σ, Σ'.
    Σ → Σ' ∧ Σ.dutyState(d) ∈ {Fulfilled, Violated}
    ⟹ Σ'.dutyState(d) = Σ.dutyState(d)
```

---

## 9. Profile Extension Points

### 9.1 Left Operands

Profiles declare operands with resolution keys:

```turtle
ex:purpose a rl15:LeftOperand ;
    rl15:contextKey "purpose" .
```

### 9.2 Action Hierarchies

```turtle
ex:read a rl15:Action ;
    rl15:includedIn ex:use .
```

### 9.3 Asset Hierarchies

```turtle
ex:customerTable a rl15:Asset ;
    rl15:partOf ex:customerSchema .
```

---

## 10. Bilateral Agreement Semantics

### 10.1 Motivation

ODRL 2.2 evaluates agreements from the assignee's perspective only. This prevents modeling provider obligations (SLAs, data quality commitments).

RL1.5 fixes this: evaluation returns duties for **both** parties.

### 10.2 Example

```turtle
ex:agreement a rl15:Agreement ;
    rl15:grantor ex:dataTeam ;
    rl15:grantee ex:analyticsTeam ;
    
    # Grantee privilege
    rl15:clause [
        a rl15:Privilege ;
        rl15:subject ex:analyticsTeam ;
        rl15:action rl15-md:display ;
        rl15:object ex:marketData
    ] ;
    
    # Grantee duty
    rl15:clause [
        a rl15:Duty ;
        rl15:subject ex:analyticsTeam ;
        rl15:action rl15-md:report ;
        rl15:object ex:usageStats ;
        rl15:deadline "P30D"^^xsd:duration
    ] ;
    
    # Grantor duty (provider SLA)
    rl15:clause [
        a rl15:Duty ;
        rl15:subject ex:dataTeam ;
        rl15:action rl15-md:notify ;
        rl15:object ex:schemaChanges ;
        rl15:deadline "P7D"^^xsd:duration
    ] .
```

Evaluation returns:
- `granteeDuties`: report usage within 30 days
- `grantorDuties`: notify of schema changes within 7 days

---
