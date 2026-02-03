# Adalbert Conformance to W3C Profile Best Practices

**Purpose**: Verify Adalbert follows W3C ODRL Profile Best Practices and W3C Profiles Vocabulary (DXPROF).

---

## W3C ODRL Profile Best Practices (CG-FINAL-profile-bp-20240808)

The W3C ODRL Community Group published best practices for ODRL 2.2 profiles. This document assesses Adalbert conformance.

---

## Best Practice Checklist

### 1. Globally Unique Identifier ✓

**Requirement**: Each profile must have a globally unique IRI identifier.

**Adalbert Compliance**:
```turtle
<https://rl2.org/adalbert/> a odrl:Profile, skos:ConceptScheme .
```

### 2. Profile as Stable Web Resource ✓

**Requirement**: Profile specification should be publicly accessible.

**Adalbert Compliance**:
- Specification: `https://rl2.org/adalbert/spec`
- Ontology: `https://rl2.org/adalbert/ontology`
- SHACL: `https://rl2.org/adalbert/shapes`

### 3. SKOS Collection Structure ✓

**Requirement**: Use SKOS Collection for profile concepts.

**Adalbert Compliance**:
```turtle
<https://rl2.org/adalbert/> a skos:ConceptScheme ;
    skos:hasTopConcept adalbert:Privilege, adalbert:Duty, adalbert:Prohibition .

<https://rl2.org/adalbert/#actions> a skos:Collection ;
    skos:prefLabel "Adalbert Actions"@en ;
    skos:member odrl:use, odrl:read, ... .

<https://rl2.org/adalbert/#leftOperands> a skos:Collection ;
    skos:prefLabel "Adalbert Left Operands"@en ;
    skos:member odrl:purpose, odrl:dateTime, ... .
```

### 4. OWL Ontology ✓

**Requirement**: Define concepts in OWL ontology.

**Adalbert Compliance**:
- File: `ontology/adalbert-core.ttl`
- Format: RDF Turtle
- Content: Class definitions, property definitions, annotation properties

### 5. Human-Readable Specification ✓

**Requirement**: Provide human-readable documentation.

**Adalbert Compliance**:
- `README.md` — Overview
- `Adalbert_Semantics.md` — Formal semantics
- `docs/*.md` — Comparison documents

### 6. W3C Profile Vocabulary (DXPROF) ✓

**Requirement**: Use W3C Profiles Vocabulary for profile description.

**Adalbert Compliance**: See [DXPROF Declaration](#dxprof-declaration) below.

### 7. JSON-LD Context ◐

**Requirement**: Provide JSON-LD context if JSON-LD supported.

**Adalbert Status**: Planned but not yet implemented.

---

## Concept Definition Requirements

### Additional Actions ✓

**Requirement**: Define new actions as subclasses of odrl:Action.

**Adalbert**: No new actions defined. Uses ODRL Core and Common Vocabulary actions.

**Note**: Domain profiles (market data, data use) add actions; Adalbert core does not.

### Additional Left Operands ✓

**Requirement**: Define new left operands.

**Adalbert**: No new left operands in core. Domain profiles add operands:
- `adalbert-md:timeliness` — Market data
- `adalbert-du:purpose` — Data use

### Additional Operators ✓

**Requirement**: Define new operators if needed.

**Adalbert**: Uses ODRL operators. Adds `adalbert:not` (missing from ODRL Core).

### Additional Party Functions ✓

**Requirement**: Define new party roles.

**Adalbert**: 
```turtle
adalbert:subject rdfs:subPropertyOf odrl:function .
adalbert:grantor rdfs:subPropertyOf odrl:assigner .
adalbert:grantee rdfs:subPropertyOf odrl:assignee .
```

### Additional Asset Relationships ✓

**Requirement**: Define new asset relations.

**Adalbert**: 
```turtle
adalbert:object rdfs:subPropertyOf odrl:relation .
```

### Rule Subclasses ✓

**Requirement**: Define rule subclasses with proper disjointness.

**Adalbert**:
```turtle
adalbert:Privilege a owl:Class ;
    rdfs:subClassOf odrl:Permission ;
    owl:disjointWith adalbert:Duty, adalbert:Prohibition .

adalbert:Duty a owl:Class ;
    rdfs:subClassOf odrl:Duty ;
    owl:disjointWith adalbert:Privilege, adalbert:Prohibition .

adalbert:Prohibition a owl:Class ;
    rdfs:subClassOf odrl:Prohibition ;
    owl:disjointWith adalbert:Privilege, adalbert:Duty .
```

### Policy Subclasses ✓

**Requirement**: Define policy subclasses with proper disjointness.

**Adalbert**:
```turtle
adalbert:Set a owl:Class ;
    rdfs:subClassOf odrl:Set ;
    owl:disjointWith odrl:Offer, odrl:Agreement .

adalbert:Agreement a owl:Class ;
    rdfs:subClassOf odrl:Agreement ;
    owl:disjointWith odrl:Offer, odrl:Set .
```

---

## DXPROF Declaration

The W3C Profiles Vocabulary ([DXPROF](https://www.w3.org/TR/dx-prof/)) provides standard metadata for profiles.

### Adalbert Profile Description

```turtle
@prefix prof: <http://www.w3.org/ns/dx/prof/> .
@prefix role: <http://www.w3.org/ns/dx/prof/role/> .
@prefix dct: <http://purl.org/dc/terms/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .

# Base specification
<http://www.w3.org/ns/odrl/2/core>
    a dct:Standard ;
    dct:title "The ODRL Core Profile Version 2.2" .

# Adalbert Profile
<https://rl2.org/adalbert/>
    a prof:Profile ;
    dct:title "Adalbert: Deterministic Rights Language"@en ;
    dct:description "A formally specified, verification-ready subset of ODRL 2.2 for data governance."@en ;
    prof:isProfileOf <http://www.w3.org/ns/odrl/2/core> ;
    prof:hasToken "adalbert" ;
    dct:publisher "RL2 Project" ;
    owl:versionInfo "0.1" ;
    dct:issued "2026-02-02"^^xsd:date ;
    
    prof:hasResource
        <https://rl2.org/adalbert/#spec> ,
        <https://rl2.org/adalbert/#semantics> ,
        <https://rl2.org/adalbert/#ontology> ,
        <https://rl2.org/adalbert/#shapes> .

# Specification document
<https://rl2.org/adalbert/#spec>
    a prof:ResourceDescriptor ;
    dct:title "Adalbert Specification"@en ;
    prof:hasRole role:specification ;
    dct:format <https://w3id.org/mediatype/text/markdown> ;
    prof:hasArtifact <https://rl2.org/adalbert/README.md> .

# Formal semantics
<https://rl2.org/adalbert/#semantics>
    a prof:ResourceDescriptor ;
    dct:title "Adalbert Formal Semantics"@en ;
    prof:hasRole role:specification ;
    dct:format <https://w3id.org/mediatype/text/markdown> ;
    prof:hasArtifact <https://rl2.org/adalbert/Adalbert_Semantics.md> .

# OWL Ontology
<https://rl2.org/adalbert/#ontology>
    a prof:ResourceDescriptor ;
    dct:title "Adalbert OWL Ontology"@en ;
    prof:hasRole role:vocabulary ;
    dct:conformsTo <https://www.w3.org/TR/owl2-rdf-based-semantics/> ;
    dct:format <https://w3id.org/mediatype/text/turtle> ;
    prof:hasArtifact <https://rl2.org/adalbert/ontology/adalbert-core.ttl> .

# SHACL Shapes
<https://rl2.org/adalbert/#shapes>
    a prof:ResourceDescriptor ;
    dct:title "Adalbert SHACL Validation Shapes"@en ;
    prof:hasRole role:validation ;
    dct:conformsTo <https://www.w3.org/TR/shacl/> ;
    dct:format <https://w3id.org/mediatype/text/turtle> ;
    prof:hasArtifact <https://rl2.org/adalbert/ontology/adalbert-shacl.ttl> .
```

### Domain Profiles

```turtle
# Market Data Profile
<https://rl2.org/adalbert/market-data>
    a prof:Profile ;
    dct:title "Adalbert Market Data Profile"@en ;
    prof:isProfileOf <https://rl2.org/adalbert/> ;
    prof:hasToken "rl15-md" ;
    prof:hasResource <https://rl2.org/adalbert/market-data#ontology> .

<https://rl2.org/adalbert/market-data#ontology>
    a prof:ResourceDescriptor ;
    prof:hasRole role:vocabulary ;
    dct:format <https://w3id.org/mediatype/text/turtle> ;
    prof:hasArtifact <https://rl2.org/adalbert/profiles/adalbert-market-data.ttl> .

# Data Use Profile
<https://rl2.org/adalbert/data-use>
    a prof:Profile ;
    dct:title "Adalbert Data Use Profile"@en ;
    prof:isProfileOf <https://rl2.org/adalbert/> ;
    prof:hasToken "rl15-du" ;
    prof:hasResource <https://rl2.org/adalbert/data-use#ontology> .

# DCON Conformance Profile
<https://rl2.org/adalbert/dcon>
    a prof:Profile ;
    dct:title "Adalbert DCON Conformance Profile"@en ;
    prof:isProfileOf <https://rl2.org/adalbert/> ;
    prof:hasToken "rl15-dcon" ;
    prof:hasResource <https://rl2.org/adalbert/dcon#ontology> .
```

---

## Profile Hierarchy

Using `prof:isTransitiveProfileOf` for the complete hierarchy:

```turtle
<https://rl2.org/adalbert/market-data>
    prof:isProfileOf <https://rl2.org/adalbert/> ;
    prof:isTransitiveProfileOf 
        <https://rl2.org/adalbert/> ,
        <http://www.w3.org/ns/odrl/2/core> .

<https://rl2.org/adalbert/data-use>
    prof:isProfileOf <https://rl2.org/adalbert/> ;
    prof:isTransitiveProfileOf 
        <https://rl2.org/adalbert/> ,
        <http://www.w3.org/ns/odrl/2/core> .
```

---

## Resource Roles Used

| Role | Adalbert Resources |
|------|-----------------|
| `role:specification` | README.md, Adalbert_Semantics.md |
| `role:vocabulary` | adalbert-core.ttl, domain profiles |
| `role:validation` | adalbert-shacl.ttl |
| `role:guidance` | docs/*.md |
| `role:example` | examples/*.ttl (planned) |

---

## Conformance Summary

| Best Practice | Status | Notes |
|---------------|--------|-------|
| Unique IRI | ✓ | `https://rl2.org/adalbert/` |
| Stable web resource | ✓ | Planned hosting |
| SKOS structure | ✓ | ConceptScheme + Collections |
| OWL ontology | ✓ | `adalbert-core.ttl` |
| Human-readable spec | ✓ | Multiple markdown docs |
| DXPROF metadata | ✓ | Full prof:Profile description |
| JSON-LD context | ◐ | Planned |
| Rule subclasses | ✓ | Proper disjointness |
| Policy subclasses | ✓ | Proper disjointness |

---

## Recommendations

### 1. Create Standalone DXPROF File

Create `ontology/adalbert-prof.ttl` with the DXPROF declarations above.

### 2. Add JSON-LD Context

Create `contexts/adalbert.jsonld`:
```json
{
  "@context": {
    "adalbert": "https://rl2.org/adalbert/",
    "odrl": "http://www.w3.org/ns/odrl/2/",
    "Privilege": "adalbert:Privilege",
    "Duty": "adalbert:Duty",
    "Prohibition": "adalbert:Prohibition",
    ...
  }
}
```

### 3. Register with ODRL Community Group

Submit profile to W3C ODRL Profiles Registry:
- Business name: Adalbert Deterministic Rights Language
- Maintainer: RL2 Project
- Identifier: `https://rl2.org/adalbert/`
- Spec URL: `https://rl2.org/adalbert/spec`
- Ontology URL: `https://rl2.org/adalbert/ontology`

---

## References

- [W3C ODRL Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
- [W3C Profiles Vocabulary (DXPROF)](https://www.w3.org/TR/dx-prof/)
- [W3C ODRL Profiles Registry](https://www.w3.org/community/odrl/wiki/ODRL_Profiles)
