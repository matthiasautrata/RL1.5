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
| `odrl:` | `http://www.w3.org/ns/odrl/2/` | Both (primary for Adalbert) |
| `prov:` | `http://www.w3.org/ns/prov#` | Both |
| `org:` | `http://www.w3.org/ns/org#` | Both |
| `skos:` | `http://www.w3.org/2004/02/skos/core#` | Both |
| `time:` | `http://www.w3.org/2006/time#` | Both |
| `adms:` | `http://www.w3.org/ns/adms#` | Both |

The `odrl:` namespace is the **primary vocabulary** for Adalbert policies. All standard rule types (`odrl:Permission`, `odrl:Duty`, `odrl:Prohibition`), policy types (`odrl:Set`, `odrl:Offer`, `odrl:Agreement`), constraint types (`odrl:Constraint`, `odrl:LogicalConstraint`), core properties (`odrl:assignee`, `odrl:assigner`, `odrl:target`, `odrl:constraint`, `odrl:action`, `odrl:leftOperand`, `odrl:operator`, `odrl:rightOperand`), and operators (`odrl:eq`, `odrl:lteq`, `odrl:and`, etc.) come from this namespace.

### DCON-Specific Namespaces

| Prefix | Namespace | Adalbert Usage |
|--------|-----------|-------------|
| `dcon:` | `https://vocabulary.bigbank/dcon/` | Referenced in `ontology/adalbert-dcon-alignment.ttl` |
| `dcon-ops-shapes:` | `https://vocabulary.bigbank/dcon/ops/shapes/` | Not used |
| `dcon-ui:` | `https://vocabulary.bigbank/dcon/ui/` | Not used |

### Adalbert-Specific Namespaces (Extensions Only)

| Prefix | Namespace | Purpose |
|--------|-----------|---------|
| `adalbert:` | `https://vocabulary.bigbank/adalbert/` | Profile extensions: State, deadline, DataContract, Subscription, partOf, memberOf, resolutionPath, RuntimeReference, currentAgent, currentDateTime, not |
| `adalbert-due:` | `https://vocabulary.bigbank/adalbert/due/` | Data use vocabulary (operands, actions, concept values) |

The `adalbert:` namespace contains **only terms that ODRL does not define**. All standard ODRL constructs are used directly from the `odrl:` namespace. The Adalbert extensions are:

- **Lifecycle**: `State`, `Pending`, `Active`, `Fulfilled`, `Violated`, `state`, `deadline`
- **Contracts**: `DataContract` (subClassOf `odrl:Offer`), `Subscription` (subClassOf `odrl:Agreement`), `subscribesTo`, `effectiveDate`, `expirationDate`
- **Hierarchy**: `partOf` (on `odrl:Asset`), `memberOf` (on `odrl:Party`)
- **Resolution**: `resolutionPath` (on `odrl:LeftOperand`)
- **Runtime**: `RuntimeReference`, `currentAgent`, `currentDateTime`
- **Logic**: `not` (on `odrl:LogicalConstraint`)

### External Vocabularies (From DCON2)

| Prefix | Namespace | Adalbert Usage |
|--------|-----------|-------------|
| `bbdp:` | `https://vocabulary.bigbank/DataPublishing/` | Could reference |
| `due:` | `https://policy.vocabulary.bigbank/due/` | Referenced in profiles |
| `dprod:` | `https://ekgf.github.io/data-product-spec/dprod/` | Not used |

### Instance Namespaces (From DCON2)

These are for ABox (instance) data, not TBox (vocabulary) definitions:

| Prefix | Namespace | Notes |
|--------|-----------|-------|
| `datacontract:` | `https://cdao.data.bigbank/dcon/` | DCON contract instances |
| `subscription:` | `https://cdao.data.bigbank/dcon/subscriptions/` | DCON subscriptions |
| `promise:` | `https://cdao.data.bigbank/dcon/promises/` | DCON promises |
| `duty:` | `https://cdao.data.bigbank/dcon/duties/` | Could be used for Adalbert duties |
| `permission:` | `https://cdao.data.bigbank/dcon/permissions/` | Could be used for Adalbert permissions |

---

## Alignment Strategy

### 1. Vocabulary Layer (TBox)

Adalbert uses ODRL terms as its primary vocabulary and defines extensions in its own namespace:
```turtle
@prefix odrl:    <http://www.w3.org/ns/odrl/2/> .       # Primary vocabulary
@prefix adalbert: <https://vocabulary.bigbank/adalbert/> . # Extensions only
```

This is distinct from DCON:
```turtle
@prefix dcon: <https://vocabulary.bigbank/dcon/> .
```

**Rationale**: Adalbert is a proper ODRL 2.2 profile; DCON is an organization-specific data contract vocabulary. They complement each other. Adalbert uses ODRL terms directly, adding only genuinely new concepts.

Contract lifecycle concepts (DataContract, Subscription) live in the `adalbert:` namespace as subclasses of ODRL types. Alignment with DCON is expressed via `skos:closeMatch` in `ontology/adalbert-dcon-alignment.ttl`.

### 2. Instance Layer (ABox)

When Adalbert policies are used within DCON contracts, instances can use DCON2 namespaces:

```turtle
# DCON subscription with Adalbert policy semantics
subscription:abc123 a dcon:DataContractSubscription ;
    dcon:hasAgreedPolicy [
        a adalbert:Subscription ;
        odrl:profile <https://vocabulary.bigbank/adalbert/> ;
        odrl:assigner ex:dataTeam ;
        odrl:assignee worker:12345 ;         # DCON2 instance namespace
        odrl:permission [
            a odrl:Permission ;
            odrl:assignee worker:12345 ;
            odrl:action odrl:derive ;
            odrl:target dataresource:xyz789      # DCON2 instance namespace
        ]
    ] .
```

### 3. Cross-References

Adalbert DCON alignment file references DCON vocabulary:
```turtle
# In ontology/adalbert-dcon-alignment.ttl
adalbert:DataContract skos:closeMatch dcon:DataContract .
adalbert:Subscription skos:closeMatch dcon:DataContractSubscription .
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

# --- Adalbert Extensions ---
@prefix adalbert:      <https://vocabulary.bigbank/adalbert/> .
@prefix adalbert-due:  <https://vocabulary.bigbank/adalbert/due/> .

# --- DCON (when integrating) ---
@prefix dcon:    <https://vocabulary.bigbank/dcon/> .
@prefix due:     <https://policy.vocabulary.bigbank/due/> .
```

---

## Namespace Design Decisions

### Decision 1: ODRL as Primary Vocabulary

**Choice**: Use `odrl:` terms directly for all standard constructs.

**Rationale**:
- Adalbert is a proper ODRL 2.2 profile, not a parallel vocabulary
- Policies are natively compatible with any ODRL processor
- The `adalbert:` namespace contains only genuinely new extensions

### Decision 2: Separate Adalbert Namespace for Extensions

**Choice**: `https://vocabulary.bigbank/adalbert/` (not under DCON)

**Rationale**:
- Adalbert is a general ODRL profile, not DCON-specific
- Enables use outside DCON context
- Follows W3C profile best practices (unique IRI)

### Decision 3: Single Profile Sub-Namespace

**Choice**: `https://vocabulary.bigbank/adalbert/due/` (data use vocabulary)

**Rationale**:
- DUE carries all vocabulary (operands, actions, concept values)
- Contract lifecycle classes (DataContract, Subscription) live in core
- DCON alignment is a mapping file, not a profile namespace

### Decision 4: Use DCON2 Instance Namespaces

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
@prefix rl2: <https://vocabulary.bigbank/> .
```

Adalbert terms would remain stable; RL2 extends them.

### 2. JSON-LD Context

Create `contexts/adalbert.jsonld` with namespace mappings for JSON-LD serialization.

---

## References

- [DCON2 Namespace Registry](../../dcon/dcon2/config/namespaces.ttl)
- [W3C ODRL Namespace](http://www.w3.org/ns/odrl/2/)
- [W3C Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
