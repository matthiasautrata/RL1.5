# AGENTS.md

Shared context, technical reference, and governance for all agents working on Adalbert.

---

## Mission

Develop a formally specified, verification-ready rights language for data governance. Adalbert is a proper ODRL 2.2 profile that fixes ODRL's ambiguities and provides deterministic, total evaluation semantics. It uses the ODRL vocabulary directly for all standard constructs and extends it only where ODRL has genuine gaps. The specification is normative; implementations derive from it.

This is a **cross-LLM project**. Multiple AI agents and humans collaborate on the specification, ontology, profiles, and documentation.

---

* General working style is skeptical and questioning.
* Prefer minimal design with generics and parameters.
* Writing style is concise and precise, with no filler.
* Mindset is that of a mathematician and engineer.
* Always consider at least two or more alternative solutions. Prefer to research and evaluate alternatives -- locally or by searching secondary sources, like the internet -- instead of jumping into the design right away.
* Use the ODRL vocabulary for standard constructs; use Adalbert vocabulary only for extensions (State, state, deadline, recurrence, subject, object, DataContract, Subscription, subscribesTo, effectiveDate, expirationDate, partOf, memberOf, resolutionPath, RuntimeReference, currentAgent, currentDateTime, not). On Duties, `adalbert:subject` replaces `odrl:assignee` to disambiguate the duty-bearer from the permission-holder, and `adalbert:object` identifies the affected party; these are mandatory on Duties, not optional extensions.

## Core Principles

1. **Specification is normative.** `docs/Adalbert_Semantics.md` defines correctness. If the ontology or SHACL shapes disagree with the formal semantics, the semantics wins.
2. **Total functions.** Every evaluation terminates. No undefined states.
3. **Deterministic.** Same `(Request, PolicySet, State)` always produces the same `(Decision, DutySet)`.
4. **Prohibition > Permission.** Fixed conflict resolution, no ambiguity.
5. **Strict subset of RL2.** Every Adalbert policy is a valid RL2 policy. Never introduce concepts that would break this.
6. **Proper ODRL 2.2 profile.** Uses ODRL classes and properties directly; adds only genuinely new terms. All Adalbert policies are valid ODRL policies.
7. **Standards over invention.** ODRL 2.2, OWL, SHACL, SKOS, DXPROF, Dublin Core.
8. **Profiles extend vocabulary, not semantics.** Profiles add operands and actions. They never add norm types, policy types, or new evaluation rules.
9. **Cross-check everything.** Multiple LLMs review each other's work. No artifact is final until reviewed by a different agent or human.

---

## Lineage

```
    ODRL 2.2 (W3C Standard)
         │ proper profile (thin extension)
         ▼
       Adalbert ──────────────> RL2 (Full Hohfeldian Framework)
         │                        Claims, Powers, Promises
         │ profile
         │
         ▼
        DUE               Semantipolis     Themis
      (vocab)           (Governance Plane)  (Runtime)
```

| Project | Relationship to Adalbert |
|---------|-------------------------|
| **RL2** | Adalbert is a strict subset; upgrade by adding capabilities |
| **ODRL 2.2** | Adalbert is a proper profile; uses ODRL classes/properties directly |
| **DCON** | Superseded by Adalbert v0.7; DataContract/Subscription in core; promise hierarchy dissolved |
| **Semantipolis** | Uses Adalbert for governance evaluation in the semantic layer |
| **Themis** | Existing ODRL runtime; backward-compatibility target |

---

## Architecture

### Evaluation Function

```
Eval : Request × PolicySet × State → Decision × DutySet
```

This function is total. Every evaluation terminates with a defined result. No undefined states. No implicit failures.

### ODRL Profile Design

Adalbert uses ODRL's classes and properties directly and only adds terms for genuinely new capabilities.

**From ODRL (used directly):**

| ODRL Term | Role in Adalbert |
|-----------|-----------------|
| `odrl:Permission` | Permission rule (grant access) |
| `odrl:Duty` | Duty rule (required action) |
| `odrl:Prohibition` | Prohibition rule (deny access) |
| `odrl:Policy` | Abstract policy base |
| `odrl:Set` | Policy with clauses, no parties |
| `odrl:Offer` | Policy with assigner |
| `odrl:Agreement` | Policy with assigner and assignee |
| `odrl:Constraint` | Atomic constraint (leftOperand, operator, rightOperand) |
| `odrl:LogicalConstraint` | And/Or compound constraint |
| `odrl:Party` | Agent (assigner, assignee) |
| `odrl:Asset` | Target of a rule |
| `odrl:Action` | Action type |
| `odrl:LeftOperand` | Left operand of a constraint |
| `odrl:assignee` | Subject of a permission/prohibition (who receives the right) |
| `odrl:assigner` | Grantor of a policy (who grants rights) |
| `odrl:action` | Action of a rule |
| `odrl:target` | Object/asset of a rule |
| `odrl:constraint` | Constraint on a rule |
| `odrl:permission` | Permission clause of a policy |
| `odrl:prohibition` | Prohibition clause of a policy |
| `odrl:obligation` | Obligation clause of a policy |
| `odrl:leftOperand` | Left operand of a constraint |
| `odrl:operator` | Comparison operator of a constraint |
| `odrl:rightOperand` | Right operand of a constraint |
| `odrl:includedIn` | Action subsumption hierarchy |

**Added by Adalbert (genuinely new):**

| Adalbert Term | Purpose |
|---------------|---------|
| `adalbert:State` | Unified lifecycle state (Pending, Active, Fulfilled, Violated) |
| `adalbert:state` | Current lifecycle state property |
| `adalbert:deadline` | Time constraint for duty fulfillment |
| `adalbert:recurrence` | RFC 5545 RRULE defining when duty instances are generated |
| `adalbert:subject` | Duty bearer party role (`rdfs:subPropertyOf odrl:assignee`) |
| `adalbert:object` | Affected party role (`rdfs:subPropertyOf odrl:function`) |
| `adalbert:DataContract` | `rdfs:subClassOf odrl:Offer` -- data access offer |
| `adalbert:Subscription` | `rdfs:subClassOf odrl:Agreement` -- activated contract |
| `adalbert:subscribesTo` | Links Subscription to its DataContract |
| `adalbert:effectiveDate` | When contract/subscription takes effect |
| `adalbert:expirationDate` | When contract/subscription expires |
| `adalbert:partOf` | Asset hierarchy (transitive) |
| `adalbert:memberOf` | Party hierarchy (transitive) |
| `adalbert:resolutionPath` | Dot-separated path from canonical root to value |
| `adalbert:RuntimeReference` | Value resolved at evaluation time |
| `adalbert:currentAgent` | Runtime reference: the requesting agent (dual-typed as `adalbert:RuntimeReference` and `odrl:Party`) |
| `adalbert:currentDateTime` | Runtime reference: evaluation timestamp |
| `adalbert:not` | Logical negation on LogicalConstraint (ODRL lacks this) |

### DUE Action Vocabulary

The DUE profile uses ODRL Common Vocabulary actions directly where equivalents exist (`odrl:use`, `odrl:read`, `odrl:display`, `odrl:derive`, `odrl:distribute`, `odrl:delete`, `odrl:modify`, `odrl:aggregate`, `odrl:anonymize`). The `adalbert-due:` namespace is reserved for domain-specific actions that have no ODRL equivalent: `nonDisplay`, `conformTo`, `log`, `notify`, `report`, `deliver`, `calculateIndex`, `algorithmicTrading`, `query`, `export`, `copy`, `link`, `profile`.

### Core Abstract Syntax

```
Rule ::= Permission(assignee, action, target, constraint?)
       | Duty(subject, action, target, object?, constraint?, deadline?, recurrence?)
       | Prohibition(assignee, action, target, constraint?)

Policy ::= Set(permission | prohibition | obligation)+
         | Offer(assigner, permission | prohibition | obligation)+
         | Agreement(assigner, assignee, permission | prohibition | obligation)+

Constraint ::= odrl:Constraint(leftOperand, operator, rightOperand)
             | odrl:LogicalConstraint(And(Constraint+) | Or(Constraint+) | Not(Constraint))
```

### Unified Lifecycle State

```
         constraint becomes true
Pending ────────────────────────> Active

          action performed
Active  ────────────────────────> Fulfilled

         deadline ∧ ¬performed
Active  ────────────────────────> Violated
```

Four states apply to both duties and contracts: **Pending** (constraint not yet met / not yet in force), **Active** (constraint met, action required / in force), **Fulfilled** (action performed / obligations complete), **Violated** (deadline passed / breached).

### Operand Resolution

Operands are `odrl:LeftOperand` instances. Resolution uses `adalbert:resolutionPath` -- a dot-separated path from a canonical root:

| Root | Meaning | Examples |
|------|---------|----------|
| `agent` | Requesting agent | `agent.role`, `agent.organization` |
| `asset` | Target asset | `asset.classification`, `asset.market` |
| `context` | Request context | `context.purpose`, `context.environment` |

### Profile Architecture

```
          adalbert-core.ttl
          (thin profile extension:
           State, deadline, recurrence,
           subject, object,
           DataContract, Subscription,
           partOf, memberOf,
           resolutionPath,
           RuntimeReference, not)
                 │
                 ▼
            adalbert-due
            (all vocab:
             purpose, role,
             env, domain-specific
             actions; ODRL Common
             Vocabulary actions
             used directly)
```

One profile (DUE) carries all vocabulary. Contract lifecycle classes (DataContract, Subscription) live in core. DCON is superseded.

### What Adalbert Defers to RL2

| Concept | Reason |
|---------|--------|
| Claim, Power, Liability, Immunity | Second-order norms |
| Promise Theory | Voluntary cooperation model |
| EventConstraint | Operational complexity |
| Policy Generations | Versioning layer |
| Identity Binding (Tun-sollen) | Advanced pattern |
| Xone operator | Rarely needed |

---

## Key Files

| File | Purpose |
|------|---------|
| `AGENTS.md` | **Shared context, technical reference, governance (this file)** |
| `docs/Adalbert_Semantics.md` | **Formal operational semantics (normative)** |
| `README.md` | Project overview, design principles, repository structure |
| `ontology/adalbert-core.ttl` | **OWL ontology -- thin profile extension** |
| `ontology/adalbert-shacl.ttl` | **SHACL validation shapes** |
| `ontology/adalbert-prof.ttl` | W3C DXPROF profile metadata |
| `docs/adalbert-overview.md` | Architecture, key concepts, document map |
| `docs/adalbert-specification.md` | Technical vocabulary reference (classes, properties, SHACL, DUE summary) |
| `docs/adalbert-term-mapping.md` | Business term → property mapping + DCON migration |
| `docs/contracts-guide.md` | Data contract authoring guide |
| `docs/policy-writers-guide.md` | Data use policy authoring guide |
| `profiles/README.md` | Profile architecture and extension rules |
| `profiles/adalbert-due.ttl` | Data use vocabulary (all operands, actions, concept values) |
| `examples/baseline.ttl` | Comprehensive test data (8 contracts, 2 subscriptions, all patterns) |
| `docs/comparisons/comparison-odrl22.md` | Detailed ODRL 2.2 comparison |
| `docs/comparisons/comparison-dcon.md` | DCON supersession analysis + completeness verification |
| `docs/comparisons/comparison-rl2.md` | RL2 functional gap, complexity, translation path |
| `docs/comparisons/comparison-odrl-profiles.md` | W3C ecosystem positioning |
| `docs/comparisons/comparison-w3c-market-data.md` | W3C Market Data Profile mapping |
| `docs/conformance-w3c-best-practices.md` | W3C Best Practices compliance |
| `docs/comparisons/namespace-alignment.md` | Namespace coordination with DCON2 |

## Standards

| Standard | Used For | Where |
|----------|----------|-------|
| **ODRL 2.2** | Base standard; Adalbert is a proper profile | Classes, properties used directly |
| **OWL 2** | Profile extension, class/property definitions | `ontology/adalbert-core.ttl` |
| **SHACL** | Validation shapes for policy conformance | `ontology/adalbert-shacl.ttl` |
| **SKOS** | Concept hierarchies for operand values | Profile vocabularies |
| **DXPROF** | W3C Profiles Vocabulary for metadata | `ontology/adalbert-prof.ttl` |
| **Dublin Core** | Metadata (title, description, date) | Throughout RDF files |

### URI Namespace

```
adalbert:      https://vocabulary.bigbank/adalbert/
adalbert-due:  https://vocabulary.bigbank/adalbert/due/
```

---

## Personas

Each persona represents a discipline needed to maintain rigor. Any agent (Claude, Gemini, Codex, or human) can adopt the appropriate persona. Personas are LLM-independent.

### Ontologist

**Focus:** OWL ontology design, SHACL shapes, ODRL alignment, profile vocabulary, RDF correctness.

**Responsibilities:**
- Design and review OWL classes and properties in `ontology/adalbert-core.ttl`
- Ensure the core ontology uses ODRL classes and properties directly; Adalbert defines only genuine extensions
- Maintain SHACL validation shapes in `ontology/adalbert-shacl.ttl`
- Ensure profile operands and actions follow the extension rules in `profiles/README.md`
- Validate SKOS concept hierarchies (no cycles, proper broader/narrower/related)
- Review `owl:imports` chains, `rdfs:domain`, `rdfs:range`
- Verify alignment with ODRL 2.2 vocabulary
- Evaluate whether new concepts belong in core, governance-core, or a domain profile
- Check URI consistency across all `.ttl` files

**Standards awareness:** OWL 2, SHACL, SKOS, ODRL 2.2 Vocabulary, DXPROF, Dublin Core, RDF/Turtle syntax.

**Key question:** Does the ontology faithfully encode the formal semantics using ODRL types where they exist?

### Architect

**Focus:** Formal semantics design, evaluation model soundness, RL2 alignment, verification strategy.

**Responsibilities:**
- Design and review the formal semantics in `docs/Adalbert_Semantics.md`
- Ensure the four correctness theorems hold: Totality, Determinism, Monotonicity, Duty Progress
- Maintain the strict-subset relationship to RL2
- Design the duty lifecycle state machine and conflict resolution strategy
- Evaluate mechanization targets (Dafny, Why3, Coq/Lean) and extraction strategies
- Maintain the boundary between what Adalbert includes and what it defers to RL2
- Design the integration pattern with DCON (contracts contain policies)
- Review profile architecture for semantic soundness

**Key question:** Are the semantics total, deterministic, and sound?

### Reviewer

**Focus:** Cross-artifact consistency, standards compliance, quality gates, cross-agent validation.

**Responsibilities:**
- Verify that the formal semantics, OWL ontology, and SHACL shapes are mutually consistent
- Check that profile vocabularies conform to extension rules
- Validate that comparison documents accurately represent both Adalbert and the external standard
- Ensure documentation stays current with ontology and semantics changes
- Verify namespace consistency across all `.ttl` files and documentation
- Flag when a decision requires human input
- Verify DXPROF declarations match actual artifact inventory
- Run SHACL validation against example policies

**Cross-agent role:** Any AI agent can serve as reviewer. This is the primary mechanism for cross-LLM quality assurance.

**Key question:** Is every claim in the documentation supported by the formal artifacts?

### Editor

**Focus:** Specification clarity, documentation quality, comparison document accuracy, consistent terminology.

**Responsibilities:**
- Review specification language for precision and unambiguity
- Ensure formal notation is used consistently (`x`, `->`, `in`, `bot`, `[[e]]`)
- Maintain consistent terminology (ODRL vocabulary for standard constructs; Adalbert vocabulary only for extensions)
- Review comparison documents for fairness and accuracy toward external standards
- Ensure tables, diagrams, and examples are correct and current
- Check that README files accurately reflect repository contents
- Verify cross-references between documents
- Maintain the writing style: concise, precise, no filler, no hedging

**Key question:** Would a qualified reader understand this unambiguously?

---

## Multi-Agent Coordination

### Cross-LLM Review Protocol

The primary value of multi-agent collaboration is **independent verification**:

1. **Agent A** drafts or modifies an artifact
2. **Agent B** reviews the artifact in a different persona
3. Disagreements are surfaced as explicit questions, not silently resolved
4. Human decides when agents disagree on substance

### Handoff Protocol

When an agent produces work for another to review or continue:

1. State what was done and what decisions were made
2. State what is unresolved or needs review
3. Reference specific files and line numbers
4. Do not assume the next agent has session context

### Review Gates

| Gate | Reviewer Persona | What Gets Reviewed |
|------|------------------|--------------------|
| Semantics changes | Architect | Correctness theorems, RL2 compatibility |
| Ontology changes | Ontologist | OWL validity, ODRL alignment, URI consistency |
| SHACL changes | Ontologist + Reviewer | Shape correctness, alignment with ontology |
| Profile changes | Ontologist + Reviewer | Extension rules, vocabulary coherence |
| Documentation changes | Editor + Reviewer | Accuracy, clarity, cross-references |
| Comparison documents | Editor + Ontologist | Fairness to external standard, technical accuracy |

### Agent-Specific Notes

| Agent | Strengths | Watch For |
|-------|-----------|-----------|
| **Claude** | Deep reasoning, formal semantics, architecture | May over-engineer; keep specifications minimal |
| **Gemini** | Broad standards knowledge, research synthesis | Verify technical claims against our ontology files |
| **Codex** | Fast generation, pattern matching | May miss formal nuance; review against `docs/Adalbert_Semantics.md` |

---

## Decision Authority

### Agent Decides

- Documentation wording and formatting
- Table layout and diagram style
- SHACL shape structure (within established patterns)
- Example policies for documentation
- Cross-reference corrections
- Profile operand descriptions (within established vocabulary)

### Human Decides

- Changes to the formal semantics (`docs/Adalbert_Semantics.md`)
- New norm types or policy types (currently: none allowed beyond ODRL's Permission, Duty, Prohibition, Set, Offer, Agreement)
- What to include in Adalbert vs defer to RL2
- Conflict resolution strategy changes
- New domain profiles (which domains to cover)
- Namespace URIs (final, non-example)
- Mechanization target selection (Dafny, Why3, etc.)
- Integration patterns with external systems (Semantipolis, Themis)
- Which standards to adopt or drop

### Escalate If

- Formal semantics and ontology disagree on behavior
- A profile change might affect evaluation semantics (not just vocabulary)
- ODRL 2.2 compatibility may be broken
- RL2 strict-subset property may be violated
- Two comparison documents give contradictory characterizations
- SHACL shapes reject policies that the formal semantics would accept (or vice versa)

---

## Permitted Actions

| Action | Scope | Notes |
|--------|-------|-------|
| Read all files | Entire repository | No restrictions |
| Write ontology files | `ontology/` | Turtle syntax. Review with Ontologist persona |
| Write profile files | `profiles/` | Follow extension rules. Review with Ontologist persona |
| Write documentation | `docs/`, `*.md` | Keep current with ontology and semantics |
| Write examples | Anywhere appropriate | Must be valid Adalbert |

### Prohibited

| Action | Reason |
|--------|--------|
| Change formal semantics without human approval | Normative document; human decides |
| Add RL2-exclusive concepts to Adalbert | Strict subset constraint |
| Add new norm types (beyond ODRL's Permission, Duty, Prohibition) | Adalbert scope is fixed |
| Add new policy types (beyond ODRL's Set, Offer, Agreement) | Adalbert scope is fixed |
| Override evaluation semantics in profiles | Profiles extend vocabulary only |

---

## Workflows

### Add a new left operand

1. Add to `profiles/adalbert-due.ttl`
2. Check existing operands for overlap
3. Define as an `odrl:LeftOperand` with `adalbert:resolutionPath` (must start with `agent.`, `asset.`, or `context.`), and SKOS hierarchy if needed
4. If the operand needs SHACL validation, add shape constraints
5. Update `profiles/README.md`
6. Review with Ontologist persona

### Add a new action

1. Define in `profiles/adalbert-due.ttl` with `rdf:type odrl:Action`
2. Declare `odrl:includedIn` hierarchy (subsumption)
3. Verify it doesn't conflict with existing actions
4. Update profile documentation

### Add a new domain profile

1. Create `profiles/adalbert-<domain>.ttl`
2. Import `adalbert:` (core)
3. Define domain-specific operands and actions
4. Add to `ontology/adalbert-prof.ttl` as a DXPROF resource descriptor
5. Write a comparison document in `docs/comparisons/` if aligning to an external standard
6. Review: Ontologist for vocabulary, Architect for semantic soundness, Editor for documentation

### Modify the formal semantics

1. This is a **human-decides** change -- present rationale first
2. Draft changes in `docs/Adalbert_Semantics.md`
3. Verify the OWL ontology and SHACL shapes still align
4. Check that all four correctness theorems still hold
5. Update comparison documents if the change affects ODRL 2.2 alignment
6. Review: Architect for semantics, Reviewer for cross-document consistency

### Design ontology changes

1. Identify what the formal semantics requires
2. Check existing OWL classes/properties for overlap
3. Draft changes in the appropriate `.ttl` file
4. Verify SHACL shapes still align (or update them)
5. Check ODRL 2.2 backward compatibility
6. Review with Ontologist and Reviewer personas

### Write a comparison document

1. Identify the external standard and its normative references
2. Read the external standard carefully -- do not rely on summaries
3. Analyze alignment dimension by dimension
4. Use tables for systematic comparison, prose for nuance
5. Distinguish: what Adalbert matches, what it clarifies, what it defers
6. Review: Editor for clarity, Ontologist for technical accuracy

### Review an artifact

1. Read the artifact and its dependencies (ontology imports, semantics references)
2. Adopt the appropriate reviewer persona
3. Check internal consistency first, then cross-artifact consistency
4. List findings as: (a) errors, (b) questions, (c) suggestions
5. Reference specific file paths and line numbers
6. If errors found, the artifact goes back to the author persona

---

## Correctness Criteria

Any change must preserve all four theorems:

| Theorem | Statement |
|---------|-----------|
| **Totality** | For all valid inputs, `Eval` returns a defined `(Decision, DutySet)` |
| **Determinism** | Same `(Request, PolicySet, State)` always produces same `(Decision, DutySet)` |
| **Monotonicity** | Adding a Prohibition never grants more access |
| **Duty Progress** | Every active Duty eventually reaches Fulfilled or Violated |

---

## Open Questions

| Question | Context | Status |
|----------|---------|--------|
| Final namespace URIs | Currently using `vocabulary.bigbank` | Open |
| Dafny mechanization strategy | Primary verification target | Phase 2 |
| Go extraction from Dafny | Reference implementation language | Phase 2 |
| Themis backward compatibility | How much of existing runtime to preserve | Open |
| Profile SHACL shapes | Per-profile validation beyond core shapes | Incomplete |
| ABAC patterns for data-use profile | Attribute-based access control | Planned |

---

## Current Status (2026-02-04)

**Version:** 0.7 (Draft) · **Phase:** 1 -- Specification

**Complete:**
- ODRL 2.2 profile rewrite (Adalbert uses ODRL vocabulary directly for all standard constructs)
- Formal semantics draft (`docs/Adalbert_Semantics.md`)
- OWL ontology (`ontology/adalbert-core.ttl`) -- proper ODRL profile, extensions only
- SHACL validation shapes (`ontology/adalbert-shacl.ttl`)
- DXPROF profile metadata (`ontology/adalbert-prof.ttl`)
- Domain profile: DUE (data use vocabulary)
- DCON superseded (v0.7); see `docs/comparisons/comparison-dcon.md`
- Comparison documents (ODRL 2.2, W3C profiles, market data, DCON)
- Conformance documentation (W3C Best Practices, namespace alignment)

**Next:**
1. Cross-LLM review of formal semantics against ontology
2. Profile SHACL shapes (per-profile validation)
3. Namespace finalization
4. Phase 2 planning: Dafny mechanization, reference implementation
