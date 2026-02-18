# Examples

Policy examples for Adalbert data-use evaluation.

## Files

| File | Purpose |
|------|---------|
| [data-use-policy.ttl](data-use-policy.ttl) | Internal access policy with role/purpose/environment constraints |

## Validation

Validate examples against SHACL:

```bash
shacl validate \
  --shapes ../ontology/adalbert-shacl.ttl \
  --data data-use-policy.ttl
```
