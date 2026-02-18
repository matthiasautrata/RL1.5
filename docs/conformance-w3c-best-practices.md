# Conformance to W3C ODRL Profile Best Practices

This document summarizes how Adalbert aligns with ODRL profile best practices.

## 1. Profile Identity

- Base standard: ODRL 2.2
- Profile IRI: `https://vocabulary.bigbank/adalbert/`
- Conflict strategy: `odrl:prohibit`
- Metadata declaration: `ontology/adalbert-prof.ttl` (DXPROF)

## 2. Reuse of ODRL Terms

Adalbert reuses ODRL classes/properties directly for standard policy modeling:

- `odrl:Permission`, `odrl:Duty`, `odrl:Prohibition`
- `odrl:Set`, `odrl:Offer`, `odrl:Agreement`
- `odrl:Constraint`, `odrl:LogicalConstraint`
- `odrl:action`, `odrl:target`, `odrl:assignee`, `odrl:assigner`, `odrl:constraint`

## 3. Extensions (Minimal)

Adalbert adds only non-ODRL capabilities:

- lifecycle state: `adalbert:State`, `adalbert:state`
- duty timing: `adalbert:deadline`, `adalbert:recurrence`
- duty roles: `adalbert:subject`, `adalbert:object`
- hierarchy: `adalbert:partOf`, `adalbert:memberOf`
- operand resolution: `adalbert:resolutionPath`
- runtime references: `adalbert:RuntimeReference`, `adalbert:currentAgent`, `adalbert:currentDateTime`
- logical negation: `adalbert:not`

## 4. Validation Artifacts

- OWL vocabulary: `ontology/adalbert-core.ttl`
- SHACL constraints: `ontology/adalbert-shacl.ttl`
- Profile metadata: `ontology/adalbert-prof.ttl`

## 5. Vocabulary Profile (DUE)

`profiles/adalbert-due.ttl` defines domain-specific operands/actions/concepts.
Where ODRL common actions already exist, Adalbert uses them directly.
