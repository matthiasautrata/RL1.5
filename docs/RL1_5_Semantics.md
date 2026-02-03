---
title: "RL1.5 Formal Semantics"
subtitle: "Deterministic Policy Evaluation for Data Governance"
version: "0.1"
status: "Draft"
date: 2026-02-02
abstract: |
  RL1.5 defines a strict subset of RL2 with deterministic, total evaluation
  semantics. This document specifies the formal semantics in a style amenable
  to future mechanization in Dafny, Why3, or similar verification frameworks.
---

## 1. Introduction

RL1.5 addresses the core semantic gaps in ODRL 2.2:

1. **Duty Ambiguity**: ODRL conflates pre-conditions with post-obligations
2. **Operational Determinism**: ODRL leaves state transitions implicit
3. **Undefined States**: ODRL permits evaluation to be "undefined"

RL1.5 fixes these by providing:

- Explicit duty lifecycle with state machine semantics
- Total evaluation functions (always terminate with defined result)
- Clear separation of Condition (pre-requisite) from Duty (obligation)

### 1.1 Notation

We use standard mathematical notation:

- `×` for Cartesian product
- `→` for function types
- `∪` for union
- `∈` for set membership
- `⊥` for undefined/bottom value
- `⟦e⟧` for denotation of expression `e`

Type judgments use `Γ ⊢ e : τ` (under context Γ, expression e has type τ).

### 1.2 Document Status

This document is **normative** for RL1.5 implementations. Any implementation
that produces different results than specified here is non-conformant.

The specification is written in a style that can be mechanized in:
- Dafny (primary target for extraction)
- Why3/WhyML (alternative with OCaml extraction)
- Coq/Lean (for independent proofs)

---

## 2. Abstract Syntax

### 2.1 Syntactic Domains

Let:

| Domain | Symbol | Description |
|--------|--------|-------------|
| Agents | **A** | Set of agent identifiers |
| Actions | **X** | Set of action identifiers |
| Assets | **S** | Set of asset identifiers |
| Values | **V** | Set of atomic values (strings, numbers, URIs) |
| Time | **T** | Time domain (ISO 8601 instants) |

### 2.2 Norms

```
Norm ::= Privilege(subject: Agent, action: Action, asset: Asset, condition: Condition?)
       | Duty(subject: Agent, action: Action, asset: Asset, condition: Condition?, deadline: Time?)
       | Prohibition(subject: Agent, action: Action, asset: Asset, condition: Condition?)
```

**Notes**:

- `Privilege` grants permission to perform an action
- `Duty` imposes an obligation to perform an action
- `Prohibition` forbids performing an action
- All norms may have optional activation conditions
- Duties may have optional deadlines

### 2.3 Conditions

```
Condition ::= AtomicConstraint(leftOperand: LeftOperand, 
                               operator: ComparisonOperator, 
                               rightOperand: Value | RuntimeRef)
            | And(operands: Condition+)
            | Or(operands: Condition+)
            | Not(operand: Condition)
```

**Operators**:

```
ComparisonOperator ::= eq | neq | lt | lte | gt | gte | isAnyOf | isNoneOf

RuntimeRef ::= currentAgent | currentTime
```

**Notes**:

- `LeftOperand` is defined by profiles (e.g., `purpose`, `classification`)
- `RuntimeRef` resolves to values at evaluation time
- `And` and `Or` require at least one operand
- `Not` requires exactly one operand

### 2.4 Policies

```
Policy ::= Set(clauses: Norm+, condition: Condition?)
         | Agreement(grantor: Agent, grantee: Agent, clauses: Norm+, condition: Condition?)
```

**Notes**:

- `Set` declares norms without specific counterparties
- `Agreement` binds identified parties
- Policy-level conditions gate all contained norms

### 2.5 Requests

```
Request ::= Request(agent: Agent, action: Action, asset: Asset, context: Context)

Context ::= Map<LeftOperand, Value>
```

A request encapsulates who wants to do what to which asset, with contextual values
for constraint evaluation.

---

## 3. Semantic Domains

### 3.1 State

State (Σ) captures the mutable aspects of the system:

```
Σ = {
    clock      : Time,
    dutyState  : Duty → DutyState,
    performed  : Set<(Agent, Action, Asset, Time)>,
    metadata   : Asset → Map<String, Value>
}
```

Where:

```
DutyState ::= Pending | Active | Fulfilled | Violated
```

**Initial state** Σ₀:

```
Σ₀ = {
    clock      = currentSystemTime,
    dutyState  = λd. Pending,
    performed  = ∅,
    metadata   = λa. {}
}
```

### 3.2 Environment

The evaluation environment bundles immutable context:

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
    decision    : Decision,
    activeDuties: Set<Duty>,
    violations  : Set<Duty>,
    explanation : Explanation
}
```

---

## 4. Duty Lifecycle

The duty lifecycle is a deterministic state machine.

### 4.1 State Diagram

```
                    condition becomes true
         ┌─────────────────────────────────────┐
         │                                     ▼
     ┌───────┐                            ┌────────┐
     │Pending│                            │ Active │
     └───────┘                            └────────┘
                                           │     │
                         action performed  │     │  deadline ∧ ¬performed
                                           ▼     ▼
                                    ┌──────────┐ ┌─────────┐
                                    │Fulfilled │ │Violated │
                                    └──────────┘ └─────────┘
```

### 4.2 Transition Rules

**Activation** (Pending → Active):

```
duty.condition ≠ ⊥ ∧ ⟦duty.condition⟧(Env) = true
────────────────────────────────────────────────
      Σ.dutyState(duty) := Active
```

**Activation** (immediate, no condition):

```
duty.condition = ⊥
─────────────────────
Σ.dutyState(duty) := Active
```

**Fulfillment** (Active → Fulfilled):

```
Σ.dutyState(duty) = Active ∧ 
(duty.subject, duty.action, duty.asset, t) ∈ Σ.performed ∧
(duty.deadline = ⊥ ∨ t ≤ duty.deadline)
────────────────────────────────────────
      Σ.dutyState(duty) := Fulfilled
```

**Violation** (Active → Violated):

```
Σ.dutyState(duty) = Active ∧
duty.deadline ≠ ⊥ ∧
Σ.clock > duty.deadline ∧
∀t. (duty.subject, duty.action, duty.asset, t) ∉ Σ.performed
────────────────────────────────────────────────────────────
      Σ.dutyState(duty) := Violated
```

### 4.3 State Transitions Are Total

For any duty `d` and state `Σ`, exactly one of the following holds:

1. `Σ.dutyState(d) = Pending` and no transition fires
2. `Σ.dutyState(d) = Pending` and Activation fires → `Active`
3. `Σ.dutyState(d) = Active` and no transition fires
4. `Σ.dutyState(d) = Active` and Fulfillment fires → `Fulfilled`
5. `Σ.dutyState(d) = Active` and Violation fires → `Violated`
6. `Σ.dutyState(d) ∈ {Fulfilled, Violated}` (terminal, no transitions)

This ensures determinism: the same inputs always produce the same state.

---

## 5. Condition Evaluation

### 5.1 Denotational Semantics

Condition evaluation is a total function:

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
    in apply(op, leftVal, rightVal)
```

**Logical connectives**:

```
⟦And(c₁, ..., cₙ)⟧(Env) = ⟦c₁⟧(Env) ∧ ... ∧ ⟦cₙ⟧(Env)

⟦Or(c₁, ..., cₙ)⟧(Env) = ⟦c₁⟧(Env) ∨ ... ∨ ⟦cₙ⟧(Env)

⟦Not(c)⟧(Env) = ¬⟦c⟧(Env)
```

### 5.2 Resolution Functions

**resolve** : LeftOperand × Env → Value

```
resolve(operand, Env) =
    if operand ∈ Env.context.keys then
        Env.context[operand]
    else if operand.resolutionPath ≠ ⊥ then
        deref(operand.resolutionPath, Env)
    else
        ⊥  // Missing operand
```

**resolveRuntime** : RuntimeRef × Env → Value

```
resolveRuntime(ref, Env) =
    case ref of
        currentAgent → Env.agent
        currentTime  → Env.state.clock
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

### 5.3 Totality

If `resolve` returns `⊥` (missing operand), the constraint evaluates to `false`:

```
⟦AtomicConstraint(left, op, right)⟧(Env) =
    let leftVal = resolve(left, Env)
    in if leftVal = ⊥ then false
       else ...
```

This ensures condition evaluation never fails—it returns `false` for malformed inputs.

---

## 6. Policy Evaluation

### 6.1 Top-Level Evaluation

```
Eval : Request × Set<Policy> × Σ → Result
```

The algorithm:

```
Eval(request, policies, Σ) =
    let Env = buildEnv(request, Σ)
    
    // 1. Find applicable policies
    let applicable = { p ∈ policies | PolicyApplicable(p, Env) }
    
    // 2. Collect matching norms
    let prohibitions = collectProhibitions(applicable, Env)
    let privileges = collectPrivileges(applicable, Env)
    let duties = collectDuties(applicable, Env)
    
    // 3. Update duty states
    let Σ' = updateDutyStates(duties, Σ)
    
    // 4. Check for active prohibitions
    if ∃p ∈ prohibitions. NormActive(p, Env) then
        return {decision: Deny, activeDuties: ∅, violations: ∅, ...}
    
    // 5. Check for granting privilege
    if ∄p ∈ privileges. NormActive(p, Env) then
        return {decision: NotApplicable, activeDuties: ∅, violations: ∅, ...}
    
    // 6. Check for violated duties
    let violated = { d ∈ duties | Σ'.dutyState(d) = Violated }
    if violated ≠ ∅ then
        return {decision: Deny, activeDuties: ∅, violations: violated, ...}
    
    // 7. Collect active duties for the response
    let active = { d ∈ duties | Σ'.dutyState(d) = Active }
    
    return {decision: Permit, activeDuties: active, violations: ∅, ...}
```

### 6.2 Policy Applicability

```
PolicyApplicable(p, Env) =
    // Asset matches
    p.target matches Env.asset ∧
    // Policy condition satisfied (if present)
    (p.condition = ⊥ ∨ ⟦p.condition⟧(Env))
```

For Agreements, also check party matching:

```
PolicyApplicable(Agreement(grantor, grantee, ...), Env) =
    ... ∧ Env.agent = grantee
```

### 6.3 Norm Activation

```
NormActive(norm, Env) =
    // Subject matches (or is universal)
    (norm.subject = Env.agent ∨ norm.subject = *) ∧
    // Action matches
    norm.action matches Env.action ∧
    // Asset matches
    norm.asset matches Env.asset ∧
    // Condition satisfied (if present)
    (norm.condition = ⊥ ∨ ⟦norm.condition⟧(Env))
```

### 6.4 Match Semantics

Action and asset matching support hierarchy subsumption:

```
action₁ matches action₂ ⟺ action₁ ⊑ action₂

asset₁ matches asset₂ ⟺ asset₁ ⊑ asset₂
```

Where `⊑` is the transitive closure of the `includedIn` relation for actions
and `partOf` relation for assets.

Profiles define the hierarchy relations.

---

## 7. Conflict Resolution

### 7.1 Default Strategy: Prohibition Overrides

RL1.5 uses a fixed conflict resolution strategy:

```
Prohibition > Privilege
```

If both a prohibition and privilege match a request, the prohibition wins.

### 7.2 Priority Within Same Norm Type

When multiple norms of the same type match, specificity determines priority:

1. More specific subject wins (named agent > role > *)
2. More specific asset wins (specific asset > collection > *)
3. More specific action wins (narrow action > broad action)

If still tied, all matching norms contribute (union for privileges, intersection for prohibitions).

---

## 8. Operational Semantics

### 8.1 Small-Step Transitions

State evolves through discrete transitions:

```
(Σ, e) → (Σ', e')
```

**Event Recording**:

```
─────────────────────────────────────────────────────
(Σ, RecordAction(agent, action, asset)) → 
(Σ[performed := Σ.performed ∪ {(agent, action, asset, Σ.clock)}], ())
```

**Clock Advance**:

```
─────────────────────────────
(Σ, AdvanceClock(t)) → (Σ[clock := t], ())
```

**Duty State Update**:

```
Σ.dutyState(d) = s ∧ transition(d, Σ) = s'
────────────────────────────────────────────
(Σ, UpdateDuty(d)) → (Σ[dutyState(d) := s'], ())
```

### 8.2 Evaluation as Transition Sequence

A complete evaluation is a sequence of transitions:

```
(Σ₀, Eval(request, policies)) 
    →* (Σₙ, Result)
```

The `→*` denotes reflexive-transitive closure.

---

## 9. Correctness Properties

### 9.1 Totality

**Theorem (Totality)**: For all well-formed inputs, `Eval` terminates with a defined result.

```
∀ request, policies, Σ.
    WellFormed(request) ∧ WellFormed(policies) ∧ WellFormed(Σ)
    ⟹ ∃ result. Eval(request, policies, Σ) = result ∧ result ≠ ⊥
```

### 9.2 Determinism

**Theorem (Determinism)**: Evaluation is deterministic.

```
∀ request, policies, Σ.
    Eval(request, policies, Σ) = r₁ ∧ Eval(request, policies, Σ) = r₂
    ⟹ r₁ = r₂
```

### 9.3 Monotonicity of Prohibition

**Theorem (Prohibition Monotonicity)**: Adding policies cannot remove prohibitions.

```
∀ request, P, P', Σ.
    P ⊆ P' ∧ Eval(request, P, Σ).decision = Deny
    ⟹ Eval(request, P', Σ).decision = Deny
```

### 9.4 Duty Lifecycle Invariants

**Theorem (Duty Progress)**: Duties can only move forward in the lifecycle.

```
∀ d, Σ, Σ'.
    Σ → Σ' ∧ Σ.dutyState(d) = Fulfilled
    ⟹ Σ'.dutyState(d) = Fulfilled

∀ d, Σ, Σ'.
    Σ → Σ' ∧ Σ.dutyState(d) = Violated
    ⟹ Σ'.dutyState(d) = Violated
```

---

## 10. Profile Extension Points

RL1.5 Core defines the evaluation framework. Profiles extend it with:

### 10.1 Left Operands

Profiles declare operands with resolution semantics:

```turtle
ex:purpose a rl15:LeftOperand ;
    rl15:contextKey "purpose" ;
    rl15:supportedOperators (rl15:eq rl15:isAnyOf) .
```

### 10.2 Actions

Profiles declare action hierarchies:

```turtle
ex:analytics a rl15:Action .
ex:loanAnalysis a rl15:Action ;
    rl15:includedIn ex:analytics .
```

### 10.3 Hierarchy Relations

Profiles may define:

- Action hierarchies via `includedIn`
- Asset hierarchies via `partOf`
- Agent hierarchies via `memberOf`

The `matches` relation uses transitive closure of these.

---

## 11. Mechanization Notes

### 11.1 Dafny Target

The semantics are structured for extraction to Dafny:

- Algebraic data types for Norm, Condition, Policy
- Functions with explicit totality proofs
- State as immutable with functional updates
- Transition rules as `ensures` clauses

### 11.2 Key Verification Goals

1. **Totality**: All functions terminate
2. **Determinism**: Same inputs → same outputs
3. **Prohibition Safety**: Prohibitions cannot be bypassed
4. **Duty Progress**: Lifecycle only moves forward

### 11.3 Implementation Extraction

Reference implementation via:

1. Dafny specification with proofs
2. Extraction to Go (via Dafny's Go backend)
3. Property-based testing against specification

---

## 12. Differences from RL2

| Aspect | RL2 | RL1.5 |
|--------|-----|-------|
| Norm types | 7 (full Hohfeld) | 3 (Privilege, Duty, Prohibition) |
| Promise Theory | Full | Deferred |
| EventConstraint | Supported | Deferred |
| Identity Binding | Tun-sollen, Sein-sollen | Deferred |
| Policy Generations | Supported | Deferred |
| Xone operator | Supported | Deferred |
| Conflict resolution | Configurable | Fixed (Prohibition > Privilege) |

RL1.5 policies are valid RL2 policies. Migration adds capabilities, not changes.

---

## 13. References

1. **RL2 Formal Semantics** — Full language specification
2. **ODRL 2.2** — W3C Recommendation (superseded semantics)
3. **Makinson & van der Torre (2000)** — Input/Output Logics
4. **Hohfeld (1913)** — Fundamental Legal Conceptions

---

*This document is normative for RL1.5 implementations.*
