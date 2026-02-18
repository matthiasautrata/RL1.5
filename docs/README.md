# Documentation

Core specification and authoring docs for the Adalbert ODRL 2.2 data-use profile.

## Core Documents

| Document | Purpose |
|----------|---------|
| [Adalbert_Semantics.md](Adalbert_Semantics.md) | Formal operational semantics (normative) |
| [adalbert-overview.md](adalbert-overview.md) | Architecture and concepts overview |
| [adalbert-specification.md](adalbert-specification.md) | Ontology and SHACL reference |
| [policy-writers-guide.md](policy-writers-guide.md) | Policy authoring guidance |
| [conformance-w3c-best-practices.md](conformance-w3c-best-practices.md) | W3C profile best-practices alignment |
| [eval-core-design-rationale.md](eval-core-design-rationale.md) | Runtime implementation rationale |

## Example Dataset

| File | Purpose |
|------|---------|
| [examples/data-use-policy.ttl](../examples/data-use-policy.ttl) | Policy-level evaluation example |

## Scope

Adalbert is a proper ODRL 2.2 profile for deterministic data-use evaluation requests.
It uses ODRL terms for standard constructs and extends only where needed:

- `adalbert:State` / `adalbert:state`
- `adalbert:deadline`
- `adalbert:recurrence`
- `adalbert:subject` / `adalbert:object`
- `adalbert:partOf` / `adalbert:memberOf`
- `adalbert:resolutionPath`
- `adalbert:RuntimeReference`, `adalbert:currentAgent`, `adalbert:currentDateTime`
- `adalbert:not`
