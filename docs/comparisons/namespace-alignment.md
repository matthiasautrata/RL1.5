# Adalbert Namespace Alignment with DCON2

**Purpose**: Document namespace coordination between Adalbert and the DCON2 namespace registry.

---

## DCON2 Namespace Registry

The authoritative namespace registry for DCON is at:
```
~/src/dcon/dcon2/config/namespaces.ttl
```

This file defines prefixes for all W3C vocabularies, DCON ontologies, external vocabularies, and instance namespaces.

---

## Namespace Categories

### W3C Standard Vocabularies (Shared)

| Prefix | Namespace | Used By |
|--------|-----------|---------|
| `rdf:` | `http://www.w3.org/1999/02/22-rdf-syntax-ns#` | Both |
| `rdfs:` | `http://www.w3.org/2000/01/rdf-schema#` | Both |
| `owl:` | `http://www.w3.org/2002/07/owl#` | Both |
| `xsd:` | `http://www.w3.org/2001/XMLSchema#` | Both |
| `sh:` | `http://www.w3.org/ns/shacl#` | Both |
| `dcat:` | `http://www.w3.org/ns/dcat#` | Both |
| `dcterms:` | `http://purl.org/dc/terms/` | Both |
| `odrl:` | `http://www.w3.org/ns/odrl/2/` | Both |
| `prov:` | `http://www.w3.org/ns/prov#` | Both |
| `org:` | `http://www.w3.org/ns/org#` | Both |
| `skos:` | `http://www.w3.org/2004/02/skos/core#` | Both |
| `time:` | `http://www.w3.org/2006/time#` | Both |
| `adms:` | `http://www.w3.org/ns/adms#` | Both |

### DCON-Specific Namespaces

| Prefix | Namespace | Adalbert Usage |
|--------|-----------|-------------|
| `dcon:` | `https://vocabulary.jpmorgan/dcon/` | Referenced in adalbert-dcon.ttl |
| `dcon-ops-shapes:` | `https://vocabulary.jpmorgan/dcon/ops/shapes/` | Not used |
| `dcon-ui:` | `https://vocabulary.jpmorgan/dcon/ui/` | Not used |

### Adalbert-Specific Namespaces (New)

| Prefix | Namespace | Purpose |
|--------|-----------|---------|
| `adalbert:` | `https://rl2.org/adalbert/` | Core vocabulary |
| `adalbert-md:` | `https://rl2.org/adalbert/market-data/` | Market data profile |
| `adalbert-du:` | `https://rl2.org/adalbert/data-use/` | Data use profile |
| `rl15-dcon:` | `https://rl2.org/adalbert/dcon/` | DCON conformance |
| `adalbert-gov:` | `https://rl2.org/adalbert/governance/` | Shared governance operands |

### External Vocabularies (From DCON2)

| Prefix | Namespace | Adalbert Usage |
|--------|-----------|-------------|
| `jpmv:` | `https://vocabulary.jpmorgan/DataPublishing/` | Could reference |
| `due:` | `https://policy.vocabulary.jpmorgan/due/` | Referenced in profiles |
| `dprod:` | `https://ekgf.github.io/data-product-spec/dprod/` | Not used |

### Instance Namespaces (From DCON2)

These are for ABox (instance) data, not TBox (vocabulary) definitions:

| Prefix | Namespace | Notes |
|--------|-----------|-------|
| `datacontract:` | `https://cdao.data.jpmorgan/dcon/` | DCON contract instances |
| `subscription:` | `https://cdao.data.jpmorgan/dcon/subscriptions/` | DCON subscriptions |
| `promise:` | `https://cdao.data.jpmorgan/dcon/promises/` | DCON promises |
| `duty:` | `https://cdao.data.jpmorgan/dcon/duties/` | Could be used for Adalbert duties |
| `permission:` | `https://cdao.data.jpmorgan/dcon/permissions/` | Could be used for Adalbert privileges |

---

## Alignment Strategy

### 1. Vocabulary Layer (TBox)

Adalbert defines its own vocabulary namespace:
```turtle
@prefix adalbert: <https://rl2.org/adalbert/> .
```

This is distinct from DCON:
```turtle
@prefix dcon: <https://vocabulary.jpmorgan/dcon/> .
```

**Rationale**: Adalbert is a general-purpose ODRL profile; DCON is an organization-specific data contract vocabulary. They complement each other.

### 2. Instance Layer (ABox)

When Adalbert policies are used within DCON contracts, instances can use DCON2 namespaces:

```turtle
# DCON subscription with Adalbert policy semantics
subscription:abc123 a dcon:DataContractSubscription ;
    dcon:hasAgreedPolicy [
        a adalbert:Agreement ;
        odrl:profile <https://rl2.org/adalbert/> ;
        adalbert:clause [
            a adalbert:Privilege ;
            adalbert:subject jpmWorker:12345 ;  # DCON2 instance namespace
            adalbert:action adalbert-du:derive ;
            adalbert:object dataresource:xyz789   # DCON2 instance namespace
        ]
    ] .
```

### 3. Cross-References

Adalbert DCON profile references DCON vocabulary:
```turtle
# In adalbert-dcon.ttl
rl15-dcon:conformsToDCON rdfs:comment "Policies using this profile are compatible with DCON DataContractSubscription."@en ;
    rdfs:seeAlso dcon:DataContractSubscription .
```

---

## Recommended Namespace Block for Adalbert Files

```turtle
# =============================================================================
# Standard Prefixes for Adalbert Files
# =============================================================================

# --- W3C Standards ---
@prefix rdf:     <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:    <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl:     <http://www.w3.org/2002/07/owl#> .
@prefix xsd:     <http://www.w3.org/2001/XMLSchema#> .
@prefix sh:      <http://www.w3.org/ns/shacl#> .
@prefix skos:    <http://www.w3.org/2004/02/skos/core#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix odrl:    <http://www.w3.org/ns/odrl/2/> .
@prefix dcat:    <http://www.w3.org/ns/dcat#> .
@prefix prov:    <http://www.w3.org/ns/prov#> .
@prefix prof:    <http://www.w3.org/ns/dx/prof/> .

# --- Adalbert Vocabularies ---
@prefix adalbert:     <https://rl2.org/adalbert/> .
@prefix adalbert-md:  <https://rl2.org/adalbert/market-data/> .
@prefix adalbert-du:  <https://rl2.org/adalbert/data-use/> .
@prefix adalbert-gov: <https://rl2.org/adalbert/governance/> .

# --- DCON (when integrating) ---
@prefix dcon:    <https://vocabulary.jpmorgan/dcon/> .
@prefix due:     <https://policy.vocabulary.jpmorgan/due/> .
```

---

## Namespace Design Decisions

### Decision 1: Separate Adalbert Namespace

**Choice**: `https://rl2.org/adalbert/` (not under DCON)

**Rationale**:
- Adalbert is a general ODRL profile, not DCON-specific
- Enables use outside DCON context
- Follows W3C profile best practices (unique IRI)

### Decision 2: Profile-Specific Sub-Namespaces

**Choice**: `https://rl2.org/adalbert/market-data/` etc.

**Rationale**:
- Clear namespace organization
- Each profile has dedicated namespace
- Follows ODRL profile convention

### Decision 3: Use DCON2 Instance Namespaces

**Choice**: Reuse `duty:`, `permission:`, etc. for instance data

**Rationale**:
- Consistent instance URIs across systems
- Avoids duplicate namespace schemes
- Enables DCON/Adalbert interoperability

---

## Future Considerations

### 1. RL2 Namespace Coordination

When RL2 is released:
```turtle
@prefix rl2: <https://rl2.org/> .
```

Adalbert terms would remain stable; RL2 extends them.

### 2. Centralized Namespace Registry

Consider creating `adalbert/config/namespaces.ttl` mirroring DCON2 pattern:
```turtle
# Adalbert Namespace Registry (Authoritative)
# Aligns with DCON2 namespace conventions
```

### 3. JSON-LD Context

Create `contexts/adalbert.jsonld` with namespace mappings for JSON-LD serialization.

---

## References

- [DCON2 Namespace Registry](../../dcon/dcon2/config/namespaces.ttl)
- [W3C ODRL Namespace](http://www.w3.org/ns/odrl/2/)
- [W3C Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
