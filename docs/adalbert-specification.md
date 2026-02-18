# Adalbert Specification

Technical reference for the Adalbert ODRL 2.2 profile extension.

## 1. Scope

Adalbert is a profile over ODRL 2.2 for deterministic data-use evaluation requests.
Standard ODRL constructs are reused directly. This specification documents only Adalbert extensions.

## 2. Extension Classes

| Term | Type | Definition |
|------|------|------------|
| `adalbert:State` | `owl:Class` | Duty lifecycle state enumeration (`Pending`, `Active`, `Fulfilled`, `Violated`) |
| `adalbert:RuntimeReference` | `owl:Class` | Value resolved at evaluation time |

## 3. Extension Properties

| Term | Type | Domain | Range | Purpose |
|------|------|--------|-------|---------|
| `adalbert:state` | `owl:ObjectProperty` | `odrl:Duty` | `adalbert:State` | Current duty lifecycle state |
| `adalbert:deadline` | `owl:DatatypeProperty` | `odrl:Duty` | `xsd:dateTime` or `xsd:duration` | Fulfillment deadline |
| `adalbert:recurrence` | `owl:DatatypeProperty` | `odrl:Duty` | `xsd:string` | RFC 5545 RRULE |
| `adalbert:subject` | `owl:ObjectProperty` | `odrl:Duty` | `odrl:Party` | Duty bearer |
| `adalbert:object` | `owl:ObjectProperty` | `odrl:Duty` | `odrl:Party` | Affected party |
| `adalbert:partOf` | `owl:ObjectProperty`, `owl:TransitiveProperty` | `odrl:Asset` | `odrl:Asset` | Asset hierarchy |
| `adalbert:memberOf` | `owl:ObjectProperty`, `owl:TransitiveProperty` | `odrl:Party` | `odrl:Party` | Party hierarchy |
| `adalbert:resolutionPath` | `owl:DatatypeProperty` | `odrl:LeftOperand` | `xsd:string` | Path resolution (`agent.*`, `asset.*`, `context.*`) |
| `adalbert:not` | `owl:ObjectProperty` | `odrl:LogicalConstraint` | `odrl:Constraint` or `odrl:LogicalConstraint` | Logical negation |

Runtime reference individuals:

- `adalbert:currentAgent`
- `adalbert:currentDateTime`

## 4. SHACL Validation Summary

Defined in `ontology/adalbert-shacl.ttl`.

| Shape | Target | Key checks |
|------|--------|------------|
| `adalbertsh:PolicyShape` | `odrl:Policy` | Correct `odrl:profile` declaration |
| `adalbertsh:SetShape` | `odrl:Set` | Non-empty clauses |
| `adalbertsh:OfferShape` | `odrl:Offer` | Assigner cardinality, non-empty clauses |
| `adalbertsh:AgreementShape` | `odrl:Agreement` | Assigner/assignee cardinality, non-empty clauses |
| `adalbertsh:PermissionShape` | `odrl:Permission` | Action/target cardinality |
| `adalbertsh:ProhibitionShape` | `odrl:Prohibition` | Action/target cardinality |
| `adalbertsh:DutyShape` | `odrl:Duty` | Action, state, deadline/recurrence types, subject/object typing |
| `adalbertsh:ConstraintShape` | `odrl:Constraint` | Left operand, operator, right operand presence |
| `adalbertsh:LogicalConstraintShape` | `odrl:LogicalConstraint` | Exactly one of `odrl:and`, `odrl:or`, `adalbert:not` |
| `adalbertsh:LeftOperandShape` | `odrl:LeftOperand` | Optional `resolutionPath` format |
| `adalbertsh:DUEOperandShape` | DUE operands | Required `resolutionPath` format |

## 5. DUE Profile

`profiles/adalbert-due.ttl` defines data-use vocabulary:

- Left operands with `adalbert:resolutionPath`
- Domain actions where ODRL has no equivalent
- SKOS concept values for right operands

ODRL common actions are used directly (for example `odrl:use`, `odrl:read`, `odrl:derive`).

## 6. Normative Source of Truth

- Semantics: `docs/Adalbert_Semantics.md`
- Ontology: `ontology/adalbert-core.ttl`
- Validation: `ontology/adalbert-shacl.ttl`
