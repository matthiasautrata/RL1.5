# Adalbert

Adalbert is a proper ODRL 2.2 profile for deterministic data-use evaluation requests.

## Design Goals

1. Deterministic evaluation for identical `(Request, PolicySet, State)` inputs
2. Total semantics (no undefined outcomes)
3. ODRL-first modeling for standard constructs
4. Minimal extensions only where ODRL has gaps

## Scope

Adalbert uses ODRL classes/properties directly for norms and policies:
`odrl:Permission`, `odrl:Duty`, `odrl:Prohibition`, `odrl:Set`, `odrl:Offer`, `odrl:Agreement`, constraints, parties, assets, actions.

Adalbert extensions:

- `adalbert:State` / `adalbert:state`
- `adalbert:deadline`
- `adalbert:recurrence`
- `adalbert:subject` / `adalbert:object`
- `adalbert:partOf` / `adalbert:memberOf`
- `adalbert:resolutionPath`
- `adalbert:RuntimeReference`, `adalbert:currentAgent`, `adalbert:currentDateTime`
- `adalbert:not`

## Repository

```text
ontology/
  adalbert-core.ttl            # OWL profile extension
  adalbert-shacl.ttl           # SHACL conformance shapes
  adalbert-prof.ttl            # DXPROF metadata

profiles/
  adalbert-due.ttl             # Data-use vocabulary (operands/actions/concepts)

examples/
  data-use-policy.ttl          # Policy evaluation example

docs/
  Adalbert_Semantics.md        # Formal semantics (normative)
  adalbert-overview.md         # Architecture overview
  adalbert-specification.md    # Vocabulary + SHACL reference
  policy-writers-guide.md      # Policy authoring guide
```

## Namespaces

| Prefix | Namespace |
|--------|-----------|
| `odrl:` | `http://www.w3.org/ns/odrl/2/` |
| `adalbert:` | `https://vocabulary.bigbank/adalbert/` |
| `adalbert-due:` | `https://vocabulary.bigbank/adalbert/due/` |

## Status

Version `0.7` draft.
