# Adalbert (RL1.5) vs RL2: Comparative Analysis

**Purpose**: Document the functional gap between Adalbert (RL1.5) and RL2, assess complexity differences, and define the translation path.

---

## Executive Summary

Adalbert is a strict subset of RL2 by design. Every Adalbert policy has a faithful RL2 representation. RL2 adds four Hohfeldian norm types, promise theory, event-based constraints, identity binding, policy generations, and a formal protocol layer. The ontology is roughly 2-3x larger; implementation complexity is 3-4x.

| Dimension | Adalbert (RL1.5) | RL2 |
|-----------|-------------------|-----|
| Norm types | 3 (Permission, Duty, Prohibition) | 7 (+Claim, Power, Liability, Immunity) |
| Policy types | 3 + 2 subtypes | 6 (Set, Offer, Agreement, Privacy, Assertion, base) |
| Condition types | 2 (Atomic, Logical) | 3 (+EventConstraint) |
| Conflict strategies | 1 (fixed: prohibition wins) | 3 (configurable) |
| Promise theory | No | Yes (Sein-sollen vs Tun-sollen) |
| Event calculus | No | Yes (typed event log, pattern matching) |
| Identity binding | No | Yes (duty performer tracking, separation of duty) |
| Policy generations | No | Yes (immutable snapshots) |
| Ontology triples | ~136 | ~656 |
| SHACL shapes | ~207 triples | ~22KB |
| Formal semantics | ~1000 lines | ~2500 lines |

---

## 1. Hohfeldian Framework (RL2 Only)

Adalbert covers first-order norms. RL2 adds the complete Hohfeldian taxonomy.

| RL2 Norm | What It Models | Adalbert Equivalent |
|----------|---------------|---------------------|
| `rl2:Privilege` | Absence of duty not to perform action | `odrl:Permission` (maps directly) |
| `rl2:Duty` | Obligation imposed on agent | `odrl:Duty` (maps directly) |
| `rl2:Prohibition` | Must not perform action | `odrl:Prohibition` (maps directly) |
| `rl2:Claim` | Entitlement held against another party | None. Implicit in duty existence but not queryable. |
| `rl2:Power` | Ability to alter normative relations | None. Cannot model delegation or revocation authority. |
| `rl2:Liability` | Exposure to another's power exercise | None. |
| `rl2:Immunity` | Protection from normative change | None. Cannot express contractual protections. |

**Correlatives modeled in RL2:**
- Duty <-> Claim (bidirectional via `rl2:correlativeTo`)
- Power <-> Liability
- Privilege <-> No-Claim (absence; not modeled as class)
- Immunity <-> Disability (inferred; not modeled as class)

**Practical impact**: Without Hohfeldian concepts, Adalbert cannot model:
- Who has authority to create, modify, or terminate policies
- Delegation chains ("the CTO can delegate data access authority to team leads")
- Contractual protections ("this access cannot be revoked during the contract period")
- Administrative vs operational authority separation

---

## 2. Promise Theory (RL2 Only)

RL2 distinguishes voluntary commitments from imposed obligations.

| Concept | RL2 | Adalbert |
|---------|-----|----------|
| **Duty (Tun-sollen)** | Imposed obligation: "you must do X" | `odrl:Duty` with `adalbert:state` |
| **Promise (Sein-sollen)** | Voluntary commitment: "I will ensure X holds" | Not modeled. Duties serve double duty. |
| **Remedial generation** | Violated promise automatically generates remedial Duty | Not supported |

**RL2 Promise lifecycle:** Pending -> Fulfilled / Violated (no Active state; promises are invariants, not action obligations).

**Practical impact**: Adalbert cannot distinguish a provider SLA guarantee ("data will be delivered by 6:30 AM") from an imposed reporting obligation ("you must submit usage reports monthly"). Both are `odrl:Duty`. This matters because:
- A violated promise should generate remedial action automatically
- A violated duty may trigger a penalty but not generate new obligations
- Promise fulfillment can be evidenced (Sein-sollen) rather than performed (Tun-sollen)

---

## 3. Event Constraints (RL2 Only)

| Feature | RL2 | Adalbert |
|---------|-----|----------|
| `rl2:EventConstraint` | "Permission requires prior approval event" | Not supported |
| Event log | Typed, time-ordered event index in state | No event tracking |
| Event sequencing | `event.after` for temporal ordering | Not supported |
| Break-glass | Event pattern triggers emergency override | Cannot model |
| Approval workflows | Multi-level approval via event chain | Cannot model |

**RL2 state includes:**
```
Events : EventType -> E*     -- Typed, temporally-ordered event index
```

**Practical impact**: Adalbert policies can only reason about current attributes (agent.role, context.purpose). They cannot express:
- "Access requires prior manager approval"
- "Permitted only after compliance attestation"
- "Emergency access allowed, but requires post-hoc review event within 24h"

---

## 4. Identity Binding (RL2 Only)

| Pattern | RL2 | Adalbert |
|---------|-----|----------|
| **Tun-sollen** | "The same agent who accessed must log" (`dutyPerformerOperand = currentAgent`) | Not expressible |
| **Separation of duty** | "Approver must not be requester" (`dutyPerformerOperand != currentAgent`) | Cannot model |
| **Performer tracking** | `DutyPerformer : Duty -> Agent` in state | Not tracked |

**Practical impact**: Adalbert cannot enforce four-eyes principles, Chinese wall policies, or audit controls that require the same (or different) agent to perform related actions.

---

## 5. Policy Generations (RL2 Only)

| Feature | RL2 | Adalbert |
|---------|-----|----------|
| Immutable generation | Complete policy snapshot; evaluation pinned to generation | Not modeled |
| Reproducibility | Given generation ID + case, result is deterministic | Deterministic per evaluation but no generation binding |
| Audit trail | Cases evaluated under generation in effect when created | `prov:wasRevisionOf` tracks lineage but no snapshot semantics |

---

## 6. Additional RL2 Capabilities

| Feature | RL2 | Adalbert |
|---------|-----|----------|
| Privacy policy type | Dedicated `rl2:Privacy` class | No dedicated type |
| Assertion policy type | `rl2:Assertion` for compliance declarations | Not modeled |
| Xone operator | Exclusive-or on logical constraints | Explicitly deferred |
| Configurable conflict | ProhibitOverrides, PermitOverrides, SpecificOverridesGeneral | Fixed prohibition-wins |
| Norm priority | Integer-based priority ordering | No priority |
| resolutionFunction | Extensible operand resolution beyond path | Path-only |
| `state.*` root | Access to event log, duty states, clock from operands | Only `currentDateTime`, `currentAgent` |
| AssetCollection | Dynamic member materialization | Not modeled |
| Protocol layer | `rl2p:Request`, `rl2p:Result`, `rl2p:Case`, `rl2p:Requirement` | Not formalized |

---

## 7. What Adalbert Preserves

Features present in both:

| Feature | Adalbert | RL2 |
|---------|----------|-----|
| Duty lifecycle | Pending -> Active -> Fulfilled / Violated | Same states (Pending -> Active -> Fulfilled / Violated) |
| Bilateral agreements | Assigner + assignee duties | Grantor + grantee duties |
| Deterministic evaluation | Total function, no undefined states | Total function with proof obligations S1-S6 |
| Hierarchy matching | `partOf` (asset), `memberOf` (agent), `includedIn` (action) | Same transitive matching |
| Operand resolution | `resolutionPath` from canonical roots (agent, asset, context) | Same + `resolutionFunction` extension |
| Logical negation | `adalbert:not` on LogicalConstraint | `rl2:not` on LogicalConstraint |
| Runtime references | `currentAgent`, `currentDateTime` | Same |
| SHACL validation | Shapes for all policy/norm types | Same pattern, more shapes |

---

## 8. Complexity Assessment

### Ontology Complexity

| Metric | Adalbert | RL2 | Factor |
|--------|----------|-----|--------|
| Core classes | 20 | ~40 | 2x |
| Properties | 27 | ~45 | 1.7x |
| Ontology triples | 136 | ~656 | 4.8x |
| SHACL shapes | 16 | ~30+ | 2x |
| Semantic domains | 10 | 15+ | 1.5x |
| Correctness theorems | 6 | 6 (S1-S6) | 1x |

### Implementation Complexity

**Adalbert evaluator requires:**
- Condition evaluation (recursive descent over atomic + logical)
- 3 norm types with simple pattern matching
- 1 conflict strategy (fixed)
- 4-state machine with 3 transitions
- Path-based operand resolution
- Transitive closure over 3 hierarchy relations

**RL2 evaluator additionally requires:**
- 4 additional norm types with correlative tracking
- Promise lifecycle (separate from duty lifecycle)
- Remedial duty generation engine
- Event constraint evaluation (pattern matching against event log)
- Event log management and temporal indexing
- Identity binding tracking (`DutyPerformer` map)
- I/O logic two-phase evaluation (monotone derivation + non-monotone resolution)
- Multiple conflict resolution strategies with priority ordering
- Policy generation management and snapshot binding
- Protocol layer (Request/Result/Case/Requirement lifecycle)
- `resolutionFunction` extension point

**Estimated implementation factor: 3-4x** for a full RL2 evaluator vs Adalbert. The Hohfeldian layer and event calculus contribute most -- they add fundamentally new evaluation mechanics, not just more types.

---

## 9. Translation: Adalbert -> RL2

Adalbert was designed as a strict subset of RL2. Translation is information-preserving (no data loss).

### Direct Mappings

| Adalbert | RL2 | Translation |
|----------|-----|-------------|
| `odrl:Permission` | `rl2:Privilege` | Type rename |
| `odrl:Duty` | `rl2:Duty` | Type rename |
| `odrl:Prohibition` | `rl2:Prohibition` | Type rename |
| `odrl:Set` | `rl2:Set` | Type rename |
| `odrl:Offer` | `rl2:Offer` | Type rename |
| `odrl:Agreement` | `rl2:Agreement` | Type rename |
| `odrl:Constraint` | `rl2:AtomicConstraint` | Type rename |
| `odrl:LogicalConstraint` | `rl2:LogicalConstraint` | Type rename |
| `odrl:assignee` (on norms) | `rl2:subject` | Property rename |
| `odrl:assigner` | `rl2:grantor` | Property rename |
| `odrl:assignee` (policy-level) | `rl2:grantee` | Property rename |
| `odrl:target` | `rl2:object` | Property rename |
| `odrl:constraint` | `rl2:condition` | Property rename |
| `odrl:action` | `rl2:action` | Direct |
| `odrl:permission` / `prohibition` / `obligation` | `rl2:clause` | Unify to single property |
| `odrl:leftOperand` | `rl2:leftOperand` | Direct |
| `odrl:operator` | `rl2:constraintOperator` | Property rename |
| `odrl:rightOperand` | `rl2:rightOperand` | Direct |
| `odrl:and` / `or` | `rl2:and` / `or` | Direct |
| `odrl:includedIn` | `rl2:includedIn` | Direct |
| `adalbert:not` | `rl2:not` | Direct |
| `adalbert:State` instances | `rl2:ObligationState` instances | Direct (same 4 states) |
| `adalbert:state` | `rl2:obligationState` | Property rename |
| `adalbert:resolutionPath` | `rl2:resolutionPath` | Direct |
| `adalbert:currentAgent` | `rl2:currentAgent` | Direct |
| `adalbert:currentDateTime` | `rl2:currentDateTime` | Direct |
| (not in Adalbert) | `rl2:rightOperandRef` | Added at RL2 (identity binding) |
| DUE operands | Profile operands | Same resolution pattern |
| DUE actions | Profile actions | Same hierarchy pattern |

### Structural Differences Requiring Transformation

1. **Deadline handling**: Adalbert has `adalbert:deadline` as a direct property on Duty. RL2 extracts deadline from conditions. A translator converts `adalbert:deadline "P30D"^^xsd:duration` into the RL2 condition pattern.

2. **DataContract / Subscription**: Adalbert defines these as subclasses. In RL2 they translate to `rl2:Offer` and `rl2:Agreement` with appropriate metadata.

3. **Conflict strategy**: Adalbert's `odrl:conflict odrl:prohibit` translates to RL2's `rl2:conflictStrategy rl2:ProhibitOverrides`.

4. **Policy-to-norm linking**: Adalbert uses typed properties (`odrl:permission`, `odrl:prohibition`, `odrl:obligation`). RL2 uses a single `rl2:clause` property. Translation unifies these.

5. **Namespace shift**: Adalbert uses `odrl:` for standard constructs; RL2 uses `rl2:` for everything. Systematic rename of every triple.

### Translation Feasibility

A mechanical translator (SPARQL CONSTRUCT or Python/rdflib script) handles ~90% of the translation. The remaining ~10% involves deadline restructuring and DataContract/Subscription type mapping.

**The translation is lossy in one direction only**: RL2 -> Adalbert loses information (Hohfeldian concepts, events, promises have no Adalbert equivalent). Adalbert -> RL2 is information-preserving.

---

## 10. Upgrade Path

The intended migration strategy:

```
Phase 1 (current): Adalbert (RL1.5)
    - First-order norms only
    - Static attribute conditions
    - Fixed conflict resolution
    - Sufficient for data-use policy and contract management

Phase 2 (future): RL2
    - Add Hohfeldian layer (Power, Claim, Immunity, Liability)
    - Add event calculus
    - Add promise theory
    - Add identity binding
    - Add policy generations
    - Existing Adalbert policies translate mechanically
```

Adalbert policies do not need rewriting -- they translate to RL2 via systematic mapping. New RL2 capabilities are additive. Organizations can adopt Adalbert now and upgrade to RL2 when they need administrative authority modeling, event-based workflows, or promise theory.

---

## References

- [RL2 Core Ontology](~/src/RL2/rl2.ttl)
- [RL2 Formal Semantics](~/src/RL2/RL2_Semantics.md)
- [RL2 Vocabulary Reference](~/src/RL2/RL2_Vocabulary.md)
- [RL2 ODRL Comparison](~/src/RL2/RL2_ODRL_Comparison.md)
- [Adalbert Formal Semantics](../Adalbert_Semantics.md)
- [Adalbert vs ODRL 2.2](comparison-odrl22.md)
