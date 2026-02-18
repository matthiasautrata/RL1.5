# Adalbert Policy Writers Guide

How to author Adalbert data-use policies.

## 1. Choose Policy Type

Use standard ODRL policy classes:

- `odrl:Set` for organization-wide policies
- `odrl:Offer` for published terms
- `odrl:Agreement` for bilateral terms

## 2. Rules

Use ODRL rule classes directly:

- `odrl:Permission`
- `odrl:Prohibition`
- `odrl:Duty`

Adalbert-specific duty roles:

- `adalbert:subject` = duty bearer
- `adalbert:object` = affected party

## 3. Constraints

Constraints follow ODRL:

- `odrl:leftOperand`
- `odrl:operator`
- `odrl:rightOperand`

Logical composition:

- `odrl:and`
- `odrl:or`
- `adalbert:not`

## 4. Duty Timing

Optional timing controls on duties:

- `adalbert:deadline` (`xsd:dateTime` or `xsd:duration`)
- `adalbert:recurrence` (RFC 5545 RRULE string)

## 5. Operand Resolution

DUE operands define `adalbert:resolutionPath` with roots:

- `agent.*`
- `asset.*`
- `context.*`

Examples:

- `context.purpose`
- `agent.role`
- `asset.classification`

## 6. Minimal Example

```turtle
@prefix ex: <http://example.org/> .
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
@prefix adalbert: <https://vocabulary.bigbank/adalbert/> .
@prefix adalbert-due: <https://vocabulary.bigbank/adalbert/due/> .

ex:policy a odrl:Set ;
    odrl:profile <https://vocabulary.bigbank/adalbert/> ;
    odrl:permission [
        a odrl:Permission ;
        odrl:assignee ex:analyst ;
        odrl:target ex:dataset ;
        odrl:action odrl:read ;
        odrl:constraint [
            a odrl:Constraint ;
            odrl:leftOperand odrl:purpose ;
            odrl:operator odrl:eq ;
            odrl:rightOperand adalbert-due:research
        ]
    ] .
```

## 7. Validation

```bash
shacl validate \
  --shapes ../ontology/adalbert-shacl.ttl \
  --data ../examples/data-use-policy.ttl
```
