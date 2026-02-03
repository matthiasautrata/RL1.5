# RL1.5 Conformance to W3C Profile Best Practices

**Purpose**: Verify RL1.5 follows W3C ODRL Profile Best Practices and W3C Profiles Vocabulary (DXPROF).

---

## W3C ODRL Profile Best Practices (CG-FINAL-profile-bp-20240808)

The W3C ODRL Community Group published best practices for ODRL 2.2 profiles. This document assesses RL1.5 conformance.

---

## Best Practice Checklist

### 1. Globally Unique Identifier ✓

**Requirement**: Each profile must have a globally unique IRI identifier.

**RL1.5 Compliance**:
```turtle
<https://rl2.org/rl1.5/> a odrl:Profile, skos:ConceptScheme .
```

### 2. Profile as Stable Web Resource ✓

**Requirement**: Profile specification should be publicly accessible.

**RL1.5 Compliance**:
- Specification: `https://rl2.org/rl1.5/spec`
- Ontology: `https://rl2.org/rl1.5/ontology`
- SHACL: `https://rl2.org/rl1.5/shapes`

### 3. SKOS Collection Structure ✓

**Requirement**: Use SKOS Collection for profile concepts.

**RL1.5 Compliance**:
```turtle
<https://rl2.org/rl1.5/> a skos:ConceptScheme ;
    skos:hasTopConcept rl15:Privilege, rl15:Duty, rl15:Prohibition .

<https://rl2.org/rl1.5/#actions> a skos:Collection ;
    skos:prefLabel "RL1.5 Actions"@en ;
    skos:member odrl:use, odrl:read, ... .

<https://rl2.org/rl1.5/#leftOperands> a skos:Collection ;
    skos:prefLabel "RL1.5 Left Operands"@en ;
    skos:member odrl:purpose, odrl:dateTime, ... .
```

### 4. OWL Ontology ✓

**Requirement**: Define concepts in OWL ontology.

**RL1.5 Compliance**:
- File: `ontology/rl1_5-core.ttl`
- Format: RDF Turtle
- Content: Class definitions, property definitions, annotation properties

### 5. Human-Readable Specification ✓

**Requirement**: Provide human-readable documentation.

**RL1.5 Compliance**:
- `README.md` — Overview
- `RL1_5_Semantics.md` — Formal semantics
- `docs/*.md` — Comparison documents

### 6. W3C Profile Vocabulary (DXPROF) ✓

**Requirement**: Use W3C Profiles Vocabulary for profile description.

**RL1.5 Compliance**: See [DXPROF Declaration](#dxprof-declaration) below.

### 7. JSON-LD Context ◐

**Requirement**: Provide JSON-LD context if JSON-LD supported.

**RL1.5 Status**: Planned but not yet implemented.

---

## Concept Definition Requirements

### Additional Actions ✓

**Requirement**: Define new actions as subclasses of odrl:Action.

**RL1.5**: No new actions defined. Uses ODRL Core and Common Vocabulary actions.

**Note**: Domain profiles (market data, data use) add actions; RL1.5 core does not.

### Additional Left Operands ✓

**Requirement**: Define new left operands.

**RL1.5**: No new left operands in core. Domain profiles add operands:
- `rl15-md:timeliness` — Market data
- `rl15-du:purpose` — Data use

### Additional Operators ✓

**Requirement**: Define new operators if needed.

**RL1.5**: Uses ODRL operators. Adds `rl15:not` (missing from ODRL Core).

### Additional Party Functions ✓

**Requirement**: Define new party roles.

**RL1.5**: 
```turtle
rl15:subject rdfs:subPropertyOf odrl:function .
rl15:grantor rdfs:subPropertyOf odrl:assigner .
rl15:grantee rdfs:subPropertyOf odrl:assignee .
```

### Additional Asset Relationships ✓

**Requirement**: Define new asset relations.

**RL1.5**: 
```turtle
rl15:object rdfs:subPropertyOf odrl:relation .
```

### Rule Subclasses ✓

**Requirement**: Define rule subclasses with proper disjointness.

**RL1.5**:
```turtle
rl15:Privilege a owl:Class ;
    rdfs:subClassOf odrl:Permission ;
    owl:disjointWith rl15:Duty, rl15:Prohibition .

rl15:Duty a owl:Class ;
    rdfs:subClassOf odrl:Duty ;
    owl:disjointWith rl15:Privilege, rl15:Prohibition .

rl15:Prohibition a owl:Class ;
    rdfs:subClassOf odrl:Prohibition ;
    owl:disjointWith rl15:Privilege, rl15:Duty .
```

### Policy Subclasses ✓

**Requirement**: Define policy subclasses with proper disjointness.

**RL1.5**:
```turtle
rl15:Set a owl:Class ;
    rdfs:subClassOf odrl:Set ;
    owl:disjointWith odrl:Offer, odrl:Agreement .

rl15:Agreement a owl:Class ;
    rdfs:subClassOf odrl:Agreement ;
    owl:disjointWith odrl:Offer, odrl:Set .
```

---

## DXPROF Declaration

The W3C Profiles Vocabulary ([DXPROF](https://www.w3.org/TR/dx-prof/)) provides standard metadata for profiles.

### RL1.5 Profile Description

```turtle
@prefix prof: <http://www.w3.org/ns/dx/prof/> .
@prefix role: <http://www.w3.org/ns/dx/prof/role/> .
@prefix dct: <http://purl.org/dc/terms/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .

# Base specification
<http://www.w3.org/ns/odrl/2/core>
    a dct:Standard ;
    dct:title "The ODRL Core Profile Version 2.2" .

# RL1.5 Profile
<https://rl2.org/rl1.5/>
    a prof:Profile ;
    dct:title "RL1.5: Deterministic Rights Language"@en ;
    dct:description "A formally specified, verification-ready subset of ODRL 2.2 for data governance."@en ;
    prof:isProfileOf <http://www.w3.org/ns/odrl/2/core> ;
    prof:hasToken "rl15" ;
    dct:publisher "RL2 Project" ;
    owl:versionInfo "0.1" ;
    dct:issued "2026-02-02"^^xsd:date ;
    
    prof:hasResource
        <https://rl2.org/rl1.5/#spec> ,
        <https://rl2.org/rl1.5/#semantics> ,
        <https://rl2.org/rl1.5/#ontology> ,
        <https://rl2.org/rl1.5/#shapes> .

# Specification document
<https://rl2.org/rl1.5/#spec>
    a prof:ResourceDescriptor ;
    dct:title "RL1.5 Specification"@en ;
    prof:hasRole role:specification ;
    dct:format <https://w3id.org/mediatype/text/markdown> ;
    prof:hasArtifact <https://rl2.org/rl1.5/README.md> .

# Formal semantics
<https://rl2.org/rl1.5/#semantics>
    a prof:ResourceDescriptor ;
    dct:title "RL1.5 Formal Semantics"@en ;
    prof:hasRole role:specification ;
    dct:format <https://w3id.org/mediatype/text/markdown> ;
    prof:hasArtifact <https://rl2.org/rl1.5/RL1_5_Semantics.md> .

# OWL Ontology
<https://rl2.org/rl1.5/#ontology>
    a prof:ResourceDescriptor ;
    dct:title "RL1.5 OWL Ontology"@en ;
    prof:hasRole role:vocabulary ;
    dct:conformsTo <https://www.w3.org/TR/owl2-rdf-based-semantics/> ;
    dct:format <https://w3id.org/mediatype/text/turtle> ;
    prof:hasArtifact <https://rl2.org/rl1.5/ontology/rl1_5-core.ttl> .

# SHACL Shapes
<https://rl2.org/rl1.5/#shapes>
    a prof:ResourceDescriptor ;
    dct:title "RL1.5 SHACL Validation Shapes"@en ;
    prof:hasRole role:validation ;
    dct:conformsTo <https://www.w3.org/TR/shacl/> ;
    dct:format <https://w3id.org/mediatype/text/turtle> ;
    prof:hasArtifact <https://rl2.org/rl1.5/ontology/rl1_5-shacl.ttl> .
```

### Domain Profiles

```turtle
# Market Data Profile
<https://rl2.org/rl1.5/market-data>
    a prof:Profile ;
    dct:title "RL1.5 Market Data Profile"@en ;
    prof:isProfileOf <https://rl2.org/rl1.5/> ;
    prof:hasToken "rl15-md" ;
    prof:hasResource <https://rl2.org/rl1.5/market-data#ontology> .

<https://rl2.org/rl1.5/market-data#ontology>
    a prof:ResourceDescriptor ;
    prof:hasRole role:vocabulary ;
    dct:format <https://w3id.org/mediatype/text/turtle> ;
    prof:hasArtifact <https://rl2.org/rl1.5/profiles/rl1_5-market-data.ttl> .

# Data Use Profile
<https://rl2.org/rl1.5/data-use>
    a prof:Profile ;
    dct:title "RL1.5 Data Use Profile"@en ;
    prof:isProfileOf <https://rl2.org/rl1.5/> ;
    prof:hasToken "rl15-du" ;
    prof:hasResource <https://rl2.org/rl1.5/data-use#ontology> .

# DCON Conformance Profile
<https://rl2.org/rl1.5/dcon>
    a prof:Profile ;
    dct:title "RL1.5 DCON Conformance Profile"@en ;
    prof:isProfileOf <https://rl2.org/rl1.5/> ;
    prof:hasToken "rl15-dcon" ;
    prof:hasResource <https://rl2.org/rl1.5/dcon#ontology> .
```

---

## Profile Hierarchy

Using `prof:isTransitiveProfileOf` for the complete hierarchy:

```turtle
<https://rl2.org/rl1.5/market-data>
    prof:isProfileOf <https://rl2.org/rl1.5/> ;
    prof:isTransitiveProfileOf 
        <https://rl2.org/rl1.5/> ,
        <http://www.w3.org/ns/odrl/2/core> .

<https://rl2.org/rl1.5/data-use>
    prof:isProfileOf <https://rl2.org/rl1.5/> ;
    prof:isTransitiveProfileOf 
        <https://rl2.org/rl1.5/> ,
        <http://www.w3.org/ns/odrl/2/core> .
```

---

## Resource Roles Used

| Role | RL1.5 Resources |
|------|-----------------|
| `role:specification` | README.md, RL1_5_Semantics.md |
| `role:vocabulary` | rl1_5-core.ttl, domain profiles |
| `role:validation` | rl1_5-shacl.ttl |
| `role:guidance` | docs/*.md |
| `role:example` | examples/*.ttl (planned) |

---

## Conformance Summary

| Best Practice | Status | Notes |
|---------------|--------|-------|
| Unique IRI | ✓ | `https://rl2.org/rl1.5/` |
| Stable web resource | ✓ | Planned hosting |
| SKOS structure | ✓ | ConceptScheme + Collections |
| OWL ontology | ✓ | `rl1_5-core.ttl` |
| Human-readable spec | ✓ | Multiple markdown docs |
| DXPROF metadata | ✓ | Full prof:Profile description |
| JSON-LD context | ◐ | Planned |
| Rule subclasses | ✓ | Proper disjointness |
| Policy subclasses | ✓ | Proper disjointness |

---

## Recommendations

### 1. Create Standalone DXPROF File

Create `ontology/rl1_5-prof.ttl` with the DXPROF declarations above.

### 2. Add JSON-LD Context

Create `contexts/rl1_5.jsonld`:
```json
{
  "@context": {
    "rl15": "https://rl2.org/rl1.5/",
    "odrl": "http://www.w3.org/ns/odrl/2/",
    "Privilege": "rl15:Privilege",
    "Duty": "rl15:Duty",
    "Prohibition": "rl15:Prohibition",
    ...
  }
}
```

### 3. Register with ODRL Community Group

Submit profile to W3C ODRL Profiles Registry:
- Business name: RL1.5 Deterministic Rights Language
- Maintainer: RL2 Project
- Identifier: `https://rl2.org/rl1.5/`
- Spec URL: `https://rl2.org/rl1.5/spec`
- Ontology URL: `https://rl2.org/rl1.5/ontology`

---

## References

- [W3C ODRL Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
- [W3C Profiles Vocabulary (DXPROF)](https://www.w3.org/TR/dx-prof/)
- [W3C ODRL Profiles Registry](https://www.w3.org/community/odrl/wiki/ODRL_Profiles)
