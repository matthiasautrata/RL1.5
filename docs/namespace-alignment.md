# RL1.5 Namespace Alignment with DCON2

**Purpose**: Document namespace coordination between RL1.5 and the DCON2 namespace registry.

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

| Prefix | Namespace | RL1.5 Usage |
|--------|-----------|-------------|
| `dcon:` | `https://vocabulary.jpmorgan/dcon/` | Referenced in rl1_5-dcon.ttl |
| `dcon-ops-shapes:` | `https://vocabulary.jpmorgan/dcon/ops/shapes/` | Not used |
| `dcon-ui:` | `https://vocabulary.jpmorgan/dcon/ui/` | Not used |

### RL1.5-Specific Namespaces (New)

| Prefix | Namespace | Purpose |
|--------|-----------|---------|
| `rl15:` | `https://rl2.org/rl1.5/` | Core vocabulary |
| `rl15-md:` | `https://rl2.org/rl1.5/market-data/` | Market data profile |
| `rl15-du:` | `https://rl2.org/rl1.5/data-use/` | Data use profile |
| `rl15-dcon:` | `https://rl2.org/rl1.5/dcon/` | DCON conformance |
| `rl15-gov:` | `https://rl2.org/rl1.5/governance/` | Shared governance operands |

### External Vocabularies (From DCON2)

| Prefix | Namespace | RL1.5 Usage |
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
| `duty:` | `https://cdao.data.jpmorgan/dcon/duties/` | Could be used for RL1.5 duties |
| `permission:` | `https://cdao.data.jpmorgan/dcon/permissions/` | Could be used for RL1.5 privileges |

---

## Alignment Strategy

### 1. Vocabulary Layer (TBox)

RL1.5 defines its own vocabulary namespace:
```turtle
@prefix rl15: <https://rl2.org/rl1.5/> .
```

This is distinct from DCON:
```turtle
@prefix dcon: <https://vocabulary.jpmorgan/dcon/> .
```

**Rationale**: RL1.5 is a general-purpose ODRL profile; DCON is an organization-specific data contract vocabulary. They complement each other.

### 2. Instance Layer (ABox)

When RL1.5 policies are used within DCON contracts, instances can use DCON2 namespaces:

```turtle
# DCON subscription with RL1.5 policy semantics
subscription:abc123 a dcon:DataContractSubscription ;
    dcon:hasAgreedPolicy [
        a rl15:Agreement ;
        odrl:profile <https://rl2.org/rl1.5/> ;
        rl15:clause [
            a rl15:Privilege ;
            rl15:subject jpmWorker:12345 ;  # DCON2 instance namespace
            rl15:action rl15-du:derive ;
            rl15:object dataresource:xyz789   # DCON2 instance namespace
        ]
    ] .
```

### 3. Cross-References

RL1.5 DCON profile references DCON vocabulary:
```turtle
# In rl1_5-dcon.ttl
rl15-dcon:conformsToDCON rdfs:comment "Policies using this profile are compatible with DCON DataContractSubscription."@en ;
    rdfs:seeAlso dcon:DataContractSubscription .
```

---

## Recommended Namespace Block for RL1.5 Files

```turtle
# =============================================================================
# Standard Prefixes for RL1.5 Files
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

# --- RL1.5 Vocabularies ---
@prefix rl15:     <https://rl2.org/rl1.5/> .
@prefix rl15-md:  <https://rl2.org/rl1.5/market-data/> .
@prefix rl15-du:  <https://rl2.org/rl1.5/data-use/> .
@prefix rl15-gov: <https://rl2.org/rl1.5/governance/> .

# --- DCON (when integrating) ---
@prefix dcon:    <https://vocabulary.jpmorgan/dcon/> .
@prefix due:     <https://policy.vocabulary.jpmorgan/due/> .
```

---

## Namespace Design Decisions

### Decision 1: Separate RL1.5 Namespace

**Choice**: `https://rl2.org/rl1.5/` (not under DCON)

**Rationale**:
- RL1.5 is a general ODRL profile, not DCON-specific
- Enables use outside DCON context
- Follows W3C profile best practices (unique IRI)

### Decision 2: Profile-Specific Sub-Namespaces

**Choice**: `https://rl2.org/rl1.5/market-data/` etc.

**Rationale**:
- Clear namespace organization
- Each profile has dedicated namespace
- Follows ODRL profile convention

### Decision 3: Use DCON2 Instance Namespaces

**Choice**: Reuse `duty:`, `permission:`, etc. for instance data

**Rationale**:
- Consistent instance URIs across systems
- Avoids duplicate namespace schemes
- Enables DCON/RL1.5 interoperability

---

## Future Considerations

### 1. RL2 Namespace Coordination

When RL2 is released:
```turtle
@prefix rl2: <https://rl2.org/> .
```

RL1.5 terms would remain stable; RL2 extends them.

### 2. Centralized Namespace Registry

Consider creating `rl1.5/config/namespaces.ttl` mirroring DCON2 pattern:
```turtle
# RL1.5 Namespace Registry (Authoritative)
# Aligns with DCON2 namespace conventions
```

### 3. JSON-LD Context

Create `contexts/rl1_5.jsonld` with namespace mappings for JSON-LD serialization.

---

## References

- [DCON2 Namespace Registry](../../dcon/dcon2/config/namespaces.ttl)
- [W3C ODRL Namespace](http://www.w3.org/ns/odrl/2/)
- [W3C Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
