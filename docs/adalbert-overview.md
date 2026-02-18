# Adalbert Overview

Adalbert is a proper ODRL 2.2 profile for data-use evaluation requests.

## Evaluation Model

`Eval : Request × PolicySet × State -> Decision × DutySet`

Adalbert requires evaluation to be total and deterministic.

## ODRL Usage

Used directly from ODRL 2.2:

- Rule classes: `odrl:Permission`, `odrl:Duty`, `odrl:Prohibition`
- Policy classes: `odrl:Set`, `odrl:Offer`, `odrl:Agreement`
- Constraints: `odrl:Constraint`, `odrl:LogicalConstraint`
- Core properties: `odrl:action`, `odrl:target`, `odrl:assignee`, `odrl:assigner`, `odrl:constraint`, `odrl:leftOperand`, `odrl:operator`, `odrl:rightOperand`

## Adalbert Extensions

- `adalbert:State` and `adalbert:state` for explicit duty lifecycle
- `adalbert:deadline` and `adalbert:recurrence` for duty timing
- `adalbert:subject` and `adalbert:object` for unambiguous duty party roles
- `adalbert:partOf` and `adalbert:memberOf` for transitive hierarchy matching
- `adalbert:resolutionPath` for deterministic operand resolution
- `adalbert:RuntimeReference`, `adalbert:currentAgent`, `adalbert:currentDateTime`
- `adalbert:not` as logical negation for `odrl:LogicalConstraint`

## Lifecycle

Duty lifecycle states:

`Pending -> Active -> Fulfilled`
`Active -> Violated` (deadline reached without performance)

## DUE Vocabulary

`profiles/adalbert-due.ttl` provides data-use-specific operands, actions, and concept values.
ODRL common vocabulary actions are reused directly where available.

## Key Artifacts

- `docs/Adalbert_Semantics.md`
- `ontology/adalbert-core.ttl`
- `ontology/adalbert-shacl.ttl`
- `profiles/adalbert-due.ttl`
- `examples/data-use-policy.ttl`
