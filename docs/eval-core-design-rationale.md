# Evaluator Core — Design Rationale

**Status:** Decision record (pre-implementation)
**Date:** 2026-02-12

---

## Problem

Adalbert needs a policy evaluator. The previous attempt (Themis) was Python, evaluating ODRL policies stored in a triple store. It worked but was neither elegant nor fast. How should we build the eval core for RL1.5, keeping the path to RL2 in mind?

## Key Insight: What the Eval Core Actually Is

The eval function is:

```
Eval : Request × PolicySet × Σ → Decision × DutySet
```

This is a **pure function**. No persistence, no event loop, no state machine. The duty lifecycle (Pending → Active → Fulfilled / Violated), case management, event sourcing, re-evaluation triggers — all of that lives in the **protocol layer** (see `RL2_Protocol.md`), outside the eval core. State enters eval as context assertions in Σ; eval doesn't mutate it.

The eval core does exactly:

1. Match request against prohibitions — any match → **Deny**
2. Collect all matching permissions
3. No matches → **Deny** (closed-world / default-deny)
4. Collect duties from matching permissions
5. Return (Decision, DutySet)

"Matching" means: the request's (agent, action, asset) satisfies a norm's (assignee, action, target) under transitive hierarchies (`includedIn`, `partOf`, `memberOf`), and all constraints hold against Σ.

## Lesson Learned: The Compile Step

Themis's mistake was interpreting ODRL triples directly at eval time — walking RDF graphs during constraint evaluation. This couples the eval core to RDF structure, producing brittle triple-pattern code that obscures the evaluation logic.

The fix: separate **compilation** (RDF → internal representation) from **evaluation** (internal representation → decision). The compiler can be complex and change with the profile vocabulary. The eval core stays small and stable, tracking only the formal semantics.

The compile step does more than reshape data. It should also **materialize transitive closures** — `includedIn⁺` (actions), `partOf⁺` (assets), `memberOf⁺` (agents) — so that eval does simple lookup, not recursive traversal. Hierarchies are static within a policy set; computing them at load time is both correct and fast.

Similarly, **recurrence expansion belongs outside eval**. The `adalbert:recurrence` property (RFC 5545 RRULE) should be pre-expanded into concrete time windows in the protocol layer before entering Σ. The eval core sees facts — "this duty instance is due between T1 and T2" — not iCal strings. This reinforces purity: eval compares values, it doesn't parse formats.

## Options Considered

### 1. Dafny eval core → Go extraction

The `design-forth-ir.md` architecture from RL2: a stack-based Forth IR, verified in Dafny, extracted to Go. Already spiked (`spike/dafny/MiniStack.dfy`).

**Rejected for RL1.5.** The Forth IR exists to serve RL2's verification needs — provable termination via fuel, bounded stack growth, minimal TCB. For RL1.5, where the eval function is simple enough to audit by inspection, this is overengineering. Additionally, the ODRL→Forth compiler is the hardest translator variant (linearizing constraint trees into stack operations, label resolution for conditionals, stack effect computation). The Forth IR belongs in the RL2 conversation.

### 2. ODRL → Prolog compiler, SWI-Prolog eval

ODRL's structure maps naturally to Prolog:

- Default deny = negation-as-failure (Prolog's native closed-world assumption)
- Prohibition > permission = clause ordering + cut
- Transitive hierarchies (`includedIn`, `partOf`, `memberOf`) = recursive clauses
- Constraint matching = unification
- Duty collection = `findall/3`
- Logical constraints (and/or/not) = direct encoding

The eval core in Prolog is roughly 70 lines. With the RDF→terms translator (~50 lines using SWI-Prolog's `semweb` library), the total is ~120 lines. Each clause corresponds to a named rule in `Adalbert_Semantics.md`.

**Concern:** No extraction path toward mechanical verification. Prolog is a dead end for Why3/Coq. If RL2 demands proofs, the eval core must be rewritten.

### 3. ODRL → Nemo (Datalog) compiler

Similar appeal to Prolog but worse fit. Datalog's stratification constraints assume monotonicity. Adalbert's prohibition-overrides-permission *is* monotonic in the right sense (adding a prohibition never grants more access), but the closed-world default-deny and `not` on `LogicalConstraint` require negation-as-failure, which pushes beyond vanilla Datalog into extensions that erode the decidability guarantees that make Datalog attractive.

**Rejected.** Impedance mismatch with the semantics.

### 4. Lean 4

Spec and implementation as the same artifact. Lean 4 is excellent for pure functions over algebraic datatypes. No extraction gap — the proved code compiles to C.

**Deferred.** Steep learning curve. Justified for RL2's complexity (Hohfeldian norms, Powers, Claims, promise-as-generator). Not justified for RL1.5's straightforward matching logic. Also, no RDF loading story — you'd still need an external compiler.

### 5. OCaml — Forth IR

The Dafny→Go path from option 1, but targeting OCaml instead (Why3 can extract to OCaml). Same objection: the Forth IR overhead is not justified for RL1.5, and the ODRL→bytecode compiler is the hardest translator variant.

**Rejected for RL1.5.** Same reasoning as option 1.

### 6. OCaml — direct interpreter

Pattern-match over ADTs, recursive eval. Same logical structure as the Prolog version but in OCaml. Estimated ~300 lines for the eval core, ~150 for the RDF→ADT translator. Total ~450 lines.

**Key advantage:** Why3 and Coq both extract to OCaml. Write the eval core in OCaml now, test rigorously, ship. Later, formalize in Why3 (spike already exists at `spike/whyml/MiniStack.mlw`), extract to OCaml, and replace the hand-written core behind the same interface. Monotonically increasing rigor, no throwaway work.

**Concern:** 4× larger than Prolog for the same logic. No native RDF loading — need an OCaml RDF library or external loader.

## Comparative Summary

| | Prolog | OCaml direct | OCaml + Forth IR |
|---|---|---|---|
| Translator | ~50 (RDF→terms) | ~150 (RDF→ADTs) | ~400 (RDF→bytecode) |
| Eval core | ~70 | ~300 | ~500 (VM) |
| **Total** | **~120** | **~450** | **~900** |
| Auditability | Highest | Good | Moderate |
| RDF loading | Free (`semweb`) | External library | External library |
| Extraction path | None | Why3/Coq → OCaml | Why3/Coq → OCaml |
| Growth to RL2 | Rewrite | Refactor | Ready |

LOC estimates are optimistic — real-world code includes logging, error handling, explanation generation, library integration. But the ratios hold directionally.

All three options beat Python by a wide margin in fit and elegance for this problem.

## Eval Boundary Contract

Regardless of path choice, the eval core presents the same interface. Define this contract now; both Prolog-via-FFI and OCaml-direct must satisfy it.

```
Eval : Request × PolicySet × Σ → (Decision × DutySet)

where
  Request  = { agent : Agent, action : Action, asset : Asset }
  Decision = Permit | PermitWithObligations | Deny
  DutySet  = Set<Duty>
  Σ        = Set<ContextAssertion>   (* facts only, no computation *)
```

The contract enforces:
- **Purity**: no side effects, no mutation of Σ
- **Totality**: always returns; no exceptions, no divergence
- **Determinism**: same inputs → same output

The protocol layer owns everything outside this boundary: persistence, event sourcing, case lifecycle, RRULE expansion, context assembly, serialization. Swapping the eval implementation — Prolog to OCaml, hand-written to extracted — changes nothing above the boundary.

## Architecture

Two viable paths. Both use the same protocol layer; they differ only in the eval core.

### Path A: Prolog eval core (recommended for RL1.5)

```
                Protocol layer (OCaml or Go)
                  │ event sourcing, case lifecycle,
                  │ context management, serialization
                  │
                  │ calls
                  ▼
Policy.ttl → [semweb loads] → RDF triples → [translate.pl] → Prolog terms → [eval.pl]
                                                                              ~70 lines
```

Three Prolog layers, cleanly separated:

| Layer | Lines | Stability |
|---|---|---|
| `eval.pl` — pure evaluation | ~70 | Stable (tracks formal semantics) |
| `translate.pl` — RDF→terms | ~50 | Changes with profile vocabulary |
| `semweb` — RDF loading | 0 (library) | SWI-Prolog maintains it |

Prolog embeds via C API or subprocess. Protocol layer remains in a systems language.

### Path B: OCaml direct (recommended if verification is a hard requirement)

```
                Protocol layer (OCaml)
                  │
                  │ calls directly
                  ▼
Policy.ttl → [RDF loader] → ADTs → [eval.ml]
                                     ~300 lines
```

Module structure:

| Module | Purpose |
|---|---|
| `types.ml` | ADTs only: Value, Norm, Policy, Request, Decision |
| `eval.ml` / `eval.mli` | Pure evaluation, locked interface |
| `translate.ml` | RDF → ADTs (profile-dependent) |

Guardrails (from prior discussion):
- Keep ADT definitions in a single module with no logic, clean seam for later extraction
- No platform-dependent `int` — use `Zarith` for all semantics-critical numeric types
- Lock behavior with property tests (from `examples/baseline.ttl`) and golden tests
- All I/O, RDF, and protocol code stays outside the eval module

### Growth path

```
RL1.5 (now)                      RL2 (later)
────────────                     ──────────
Path A: Prolog eval core    →    Rewrite to OCaml (or keep Prolog
        ~120 lines                if complexity stays manageable)

Path B: OCaml eval core     →    Why3 formalization + extraction
        ~450 lines                replaces hand-written eval.ml
                                  (spike exists: spike/whyml/)
```

Both paths converge on OCaml for RL2 if mechanical verification is pursued.

## Open Questions

| Question | Context |
|---|---|
| Protocol layer language | OCaml (unified with Path B) or Go (cloud-native)? |
| RDF loading for OCaml path | Which library? Or shell out to `rapper`/`riot`? |
| Prolog embedding strategy | C API (`libswipl`) vs subprocess vs socket? |
| Test oracle | Generate golden tests from `examples/baseline.ttl` before building? |
| Path choice | Depends on how seriously we weight verification for RL1.5 |

## References

- `docs/Adalbert_Semantics.md` — formal semantics (normative)
- `RL2/RL2_Protocol.md` — protocol layer, case lifecycle, event sourcing
- `RL2/design-forth-ir.md` — Forth IR design (RL2 scope)
- `RL2/spike/` — Dafny, Stainless, WhyML verification spikes
- `RL2/spike/whyml/MiniStack.mlw` — Why3 spike
