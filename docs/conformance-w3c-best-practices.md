# Adalbert Conformance to W3C Profile Best Practices

**Purpose**: Verify Adalbert follows W3C ODRL Profile Best Practices and W3C Profiles Vocabulary (DXPROF).

---

## W3C ODRL Profile Best Practices (CG-FINAL-profile-bp-20240808)

The W3C ODRL Community Group published best practices for ODRL 2.2 profiles. This document assesses Adalbert conformance.

---

## Best Practice Checklist

### 1. Globally Unique Identifier

**Requirement**: Each profile must have a globally unique IRI identifier.

**Adalbert Compliance**:
```turtle
<https://vocabulary.bigbank/adalbert/> a odrl:Profile, skos:ConceptScheme .
```

### 2. Profile as Stable Web Resource

**Requirement**: Profile specification should be publicly accessible.

**Adalbert Compliance**:
- Specification: `https://vocabulary.bigbank/adalbert/spec`
- Ontology: `https://vocabulary.bigbank/adalbert/ontology`
- SHACL: `https://vocabulary.bigbank/adalbert/shapes`

### 3. SKOS Collection Structure

**Requirement**: Use SKOS Collection for profile concepts.

**Adalbert Compliance**: Adalbert uses ODRL types directly (not its own rule types), so SKOS collections reference ODRL classes:
```turtle
<https://vocabulary.bigbank/adalbert/> a skos:ConceptScheme ;
    skos:hasTopConcept odrl:Permission, odrl:Duty, odrl:Prohibition .

<https://vocabulary.bigbank/adalbert/#actions> a skos:Collection ;
    skos:prefLabel "Adalbert Actions"@en ;
    skos:member odrl:use, odrl:read, ... .

<https://vocabulary.bigbank/adalbert/#leftOperands> a skos:Collection ;
    skos:prefLabel "Adalbert Left Operands"@en ;
    skos:member odrl:purpose, odrl:dateTime, ... .

<https://vocabulary.bigbank/adalbert/#extensions> a skos:Collection ;
    skos:prefLabel "Adalbert Extensions"@en ;
    skos:member adalbert:State, adalbert:DataContract,
               adalbert:Subscription, adalbert:deadline,
               adalbert:not .
```

### 4. OWL Ontology

**Requirement**: Define concepts in OWL ontology.

**Adalbert Compliance**:
- File: `ontology/adalbert-core.ttl`
- Format: RDF Turtle
- Content: Extension class definitions (State, DataContract, Subscription), extension property definitions (state, deadline, partOf, memberOf, resolutionPath, etc.), annotation properties

### 5. Human-Readable Specification

**Requirement**: Provide human-readable documentation.

**Adalbert Compliance**:
- `README.md` -- Overview
- `docs/Adalbert_Semantics.md` -- Formal semantics
- `docs/*.md` -- Comparison documents

### 6. W3C Profile Vocabulary (DXPROF)

**Requirement**: Use W3C Profiles Vocabulary for profile description.

**Adalbert Compliance**: See [DXPROF Declaration](#dxprof-declaration) below.

### 7. JSON-LD Context (Partial)

**Requirement**: Provide JSON-LD context if JSON-LD supported.

**Adalbert Status**: Planned but not yet implemented.

---

## Concept Definition Requirements

### Additional Actions

**Requirement**: Define new actions as subclasses of `odrl:Action`.

**Adalbert**: No new actions defined in core. Uses ODRL Core and Common Vocabulary actions. Domain profiles (DUE) add actions.

### Additional Left Operands

**Requirement**: Define new left operands.

**Adalbert**: No new left operands in core. One runtime reference (`adalbert:currentDateTime`) has `skos:exactMatch odrl:dateTime`. Domain profiles add operands:
- `adalbert-due:timeliness` -- Data use (market data)
- `adalbert-due:purpose` -- Data use

### Additional Operators

**Requirement**: Define new operators if needed.

**Adalbert**: Uses ODRL operators directly (`odrl:eq`, `odrl:neq`, `odrl:lt`, `odrl:lteq`, `odrl:gt`, `odrl:gteq`, `odrl:isAnyOf`, `odrl:isNoneOf`, `odrl:and`, `odrl:or`). Adds `adalbert:not` (logical negation, missing from ODRL Core).

### Additional Party Functions

**Requirement**: Define new party roles.

**Adalbert**: Uses `odrl:assignee` and `odrl:assigner` directly. No sub-properties defined. Hierarchy is expressed via `adalbert:memberOf` (transitive party membership on `odrl:Party`).

### Additional Asset Relationships

**Requirement**: Define new asset relations.

**Adalbert**: Uses `odrl:target` directly. Adds `adalbert:partOf` (transitive asset containment on `odrl:Asset`).

### Rule Types

**Requirement**: Define rule subclasses with proper disjointness if needed.

**Adalbert**: Does not define rule subclasses. Uses ODRL rule types directly:
- `odrl:Permission` -- used as-is
- `odrl:Duty` -- used as-is, with added `adalbert:state` and `adalbert:deadline` properties
- `odrl:Prohibition` -- used as-is

### Policy Subclasses

**Requirement**: Define policy subclasses with proper disjointness.

**Adalbert**: Defines two domain-specific policy subclasses:
```turtle
adalbert:DataContract a owl:Class ;
    rdfs:subClassOf odrl:Offer ;
    rdfs:label "Data Contract"@en ;
    skos:definition "Policy offer defining data access terms."@en .

adalbert:Subscription a owl:Class ;
    rdfs:subClassOf odrl:Agreement ;
    rdfs:label "Subscription"@en ;
    skos:definition "Activated data contract binding provider and consumer."@en .
```

`odrl:Set` is used directly without subclassing.

---

## DXPROF Declaration

The W3C Profiles Vocabulary ([DXPROF](https://www.w3.org/TR/dx-prof/)) provides standard metadata for profiles.

### Adalbert Profile Description

From `ontology/adalbert-prof.ttl`:

```turtle
@prefix prof: <http://www.w3.org/ns/dx/prof/> .
@prefix role: <http://www.w3.org/ns/dx/prof/role/> .
@prefix dct:  <http://purl.org/dc/terms/> .
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
@prefix owl:  <http://www.w3.org/2002/07/owl#> .

# Base specification
odrl:core a dct:Standard ;
    dct:title "ODRL Information Model 2.2"@en ;
    dct:identifier <http://www.w3.org/TR/odrl-model/> .

# Adalbert Core Profile
<https://vocabulary.bigbank/adalbert/> a prof:Profile ;
    dct:title "Adalbert Core"@en ;
    dct:description """
        Proper ODRL 2.2 profile for deterministic data governance.
        Uses ODRL terms for standard constructs; adds lifecycle state,
        deadline, data contracts, hierarchy extensions, operand resolution,
        runtime references, and logical negation.
    """@en ;
    prof:isProfileOf odrl:core ;
    prof:hasToken "adalbert"^^xsd:token ;

    prof:hasResource [
        a prof:ResourceDescriptor ;
        dct:title "Adalbert Specification"@en ;
        prof:hasRole role:specification ;
        prof:hasArtifact <https://vocabulary.bigbank/adalbert/docs/README.md>
    ] , [
        a prof:ResourceDescriptor ;
        dct:title "Adalbert Formal Semantics"@en ;
        prof:hasRole role:specification ;
        prof:hasArtifact <https://vocabulary.bigbank/adalbert/docs/Adalbert_Semantics.md>
    ] , [
        a prof:ResourceDescriptor ;
        dct:title "Adalbert Core Ontology"@en ;
        prof:hasRole role:vocabulary ;
        dct:conformsTo owl: ;
        prof:hasArtifact <https://vocabulary.bigbank/adalbert/ontology/adalbert-core.ttl>
    ] , [
        a prof:ResourceDescriptor ;
        dct:title "Adalbert SHACL Shapes"@en ;
        prof:hasRole role:validation ;
        dct:conformsTo <http://www.w3.org/ns/shacl#> ;
        prof:hasArtifact <https://vocabulary.bigbank/adalbert/ontology/adalbert-shacl.ttl>
    ] .
```

### Domain Profiles

```turtle
# Data Use Profile
<https://vocabulary.bigbank/adalbert/due/> a prof:Profile ;
    dct:title "Adalbert Data Use (DUE)"@en ;
    dct:description "Complete data governance vocabulary: operands, actions, concept values."@en ;
    prof:isProfileOf <https://vocabulary.bigbank/adalbert/> ;
    prof:isTransitiveProfileOf odrl:core ;
    prof:hasToken "adalbert-due"^^xsd:token ;

    prof:hasResource [
        a prof:ResourceDescriptor ;
        dct:title "Adalbert DUE Vocabulary"@en ;
        prof:hasRole role:vocabulary ;
        dct:conformsTo owl: ;
        prof:hasArtifact <https://vocabulary.bigbank/adalbert/profiles/adalbert-due.ttl>
    ] .
```

---

## Profile Hierarchy

Using `prof:isTransitiveProfileOf` for the complete hierarchy:

```turtle
<https://vocabulary.bigbank/adalbert/due/>
    prof:isProfileOf <https://vocabulary.bigbank/adalbert/> ;
    prof:isTransitiveProfileOf
        <https://vocabulary.bigbank/adalbert/> ,
        odrl:core .
```

---

## Resource Roles Used

| Role | Adalbert Resources |
|------|-----------------|
| `role:specification` | README.md, Adalbert_Semantics.md |
| `role:vocabulary` | adalbert-core.ttl, adalbert-due.ttl |
| `role:validation` | adalbert-shacl.ttl |
| `role:guidance` | docs/*.md |
| `role:example` | examples/*.ttl |

---

## Conformance Summary

| Best Practice | Status | Notes |
|---------------|--------|-------|
| Unique IRI | Done | `https://vocabulary.bigbank/adalbert/` |
| Stable web resource | Done | Planned hosting |
| SKOS structure | Done | ConceptScheme + Collections referencing ODRL types |
| OWL ontology | Done | `adalbert-core.ttl` (extensions only) |
| Human-readable spec | Done | Multiple markdown docs |
| DXPROF metadata | Done | `adalbert-prof.ttl` |
| JSON-LD context | Planned | Not yet implemented |
| Rule types | Done | Uses ODRL types directly (no subclasses) |
| Policy subclasses | Done | DataContract (Offer), Subscription (Agreement) |

---

## Recommendations

### 1. Add JSON-LD Context

Create `contexts/adalbert.jsonld`:
```json
{
  "@context": {
    "adalbert": "https://vocabulary.bigbank/adalbert/",
    "odrl": "http://www.w3.org/ns/odrl/2/",
    "Permission": "odrl:Permission",
    "Duty": "odrl:Duty",
    "Prohibition": "odrl:Prohibition",
    "DataContract": "adalbert:DataContract",
    "Subscription": "adalbert:Subscription",
    "state": "adalbert:state",
    "deadline": "adalbert:deadline"
  }
}
```

### 2. Register with ODRL Community Group

Submit profile to W3C ODRL Profiles Registry:
- Business name: Adalbert Deterministic Rights Language
- Maintainer: RL2 Project
- Identifier: `https://vocabulary.bigbank/adalbert/`
- Spec URL: `https://vocabulary.bigbank/adalbert/spec`
- Ontology URL: `https://vocabulary.bigbank/adalbert/ontology`

---

## References

- [W3C ODRL Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
- [W3C Profiles Vocabulary (DXPROF)](https://www.w3.org/TR/dx-prof/)
- [W3C ODRL Profiles Registry](https://www.w3.org/community/odrl/wiki/ODRL_Profiles)
