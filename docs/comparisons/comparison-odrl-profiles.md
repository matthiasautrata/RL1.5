# Adalbert and the W3C ODRL Profile Ecosystem

**Purpose**: Position Adalbert within the broader landscape of ODRL 2.2 profiles.

---

## Overview

The W3C ODRL Community Group maintains a [registry of ODRL profiles](https://www.w3.org/community/odrl/wiki/ODRL_Profiles). Adalbert is a proper ODRL 2.2 profile that provides semantic precision and formal verification while remaining compatible with domain-specific profiles.

---

## Known ODRL Profiles

| Profile | Domain | Primary Focus | Relationship to Adalbert |
|---------|--------|---------------|----------------------|
| **Data Sovereignty** | Data governance | Cross-border data flows | Complementary |
| **RightsML** | Media/publishing | News syndication rights | Separate domain |
| **Market Data** | Financial services | Market data licensing | Adalbert has aligned profile |
| **Big Data** | Data processing | Large-scale data rights | Complementary |
| **Access Control** | Security | Resource authorization | Adalbert has aligned profile |
| **VC-based** | Identity | Verifiable credentials | Complementary |
| **Language Resources** | NLP/AI | Linguistic data licensing | Separate domain |
| **Regulatory Compliance** | Legal | Policy compliance | Complementary |
| **Privacy Paradigm** | Privacy | Data protection rights | Complementary |
| **ToS Language** | Legal | Terms of service | Separate domain |

---

## Adalbert's Position

### What Adalbert Is

Adalbert is a **proper ODRL 2.2 profile** that uses ODRL terms directly and adds extensions for deterministic evaluation. It:

1. **Uses ODRL vocabulary** for all standard constructs (Permission, Duty, Prohibition, Constraint, etc.)
2. **Adds formal operational semantics** for verification
3. **Extends with lifecycle tracking** (State, deadline) for duties and contracts
4. **Enables domain profiles** to inherit deterministic behavior

### What Adalbert Is Not

- Not a domain-specific vocabulary
- Not a replacement for existing profiles
- Not a parallel vocabulary (it uses ODRL terms, not its own equivalents)

---

## Architectural Relationship

```
                    +-------------------------------------+
                    |         ODRL 2.2 Core               |
                    |    (W3C Recommendation)              |
                    +----------------+--------------------+
                                     |
           +-------------------------+-------------------------+
           |                         |                         |
           v                         v                         v
    +--------------+         +--------------+         +--------------+
    |   Adalbert   |         |  RightsML    |         | Data Sov.    |
    | (Profile:    |         |  (Domain)    |         |  (Domain)    |
    |  Semantic    |         |              |         |              |
    |  Foundation) |         +--------------+         +--------------+
    +------+-------+
           |
    +------+-------+
    |              |
    v              v
+--------+  +----------+
|  Data  |  |   DCON   |
|  Use   |  | Alignment|
|Profile |  |          |
+--------+  +----------+
```

**Key insight**: Adalbert sits between ODRL 2.2 Core and domain profiles, providing semantic precision that domain profiles can inherit. Because Adalbert uses ODRL terms directly, policies are natively interoperable with the broader ODRL ecosystem.

---

## Comparison with Major Profiles

### Data Sovereignty Profile

**Focus**: Cross-border data transfer rules, jurisdiction constraints.

| Aspect | Data Sovereignty | Adalbert |
|--------|------------------|-------|
| Purpose | Geographic policy | Semantic foundation |
| Adds actions | Yes (transfer, localize) | No |
| Adds operands | Yes (jurisdiction, region) | No (via DUE profile) |
| Formal semantics | No | Yes |
| Verification target | No | Yes |

**Integration**: Data sovereignty constraints can be expressed as ODRL constraints using domain-specific operands. An Adalbert-aware processor enforces deterministic evaluation:

```turtle
# Data Sovereignty concern expressed in an Adalbert policy
ex:rule a odrl:Permission ;
    odrl:constraint [
        a odrl:Constraint ;
        odrl:leftOperand ds:dataLocation ;
        odrl:operator odrl:isAnyOf ;
        odrl:rightOperand ( ds:EU ds:UK ds:CH )
    ] .
```

### RightsML

**Focus**: News content syndication and media rights.

| Aspect | RightsML | Adalbert |
|--------|----------|-------|
| Domain | Media/publishing | Cross-domain |
| Vocabulary | Rich (100+ terms) | Minimal extensions |
| Payment modeling | Yes | No (deferred) |
| Formal semantics | No | Yes |

**Integration**: RightsML is domain-specific. Adalbert would not replace it but could provide underlying semantic clarity if RightsML policies needed formal verification.

### Access Control Profile

**Focus**: Authentication, authorization, resource access.

| Aspect | Access Control | Adalbert Data Use Profile |
|--------|---------------|------------------------|
| Focus | Technical access | Business authorization |
| Actions | authenticate, authorize | derive, distribute |
| Party model | User, Group, Role | Internal organization |
| Formal semantics | No | Yes |

**Integration**: The profiles address different concerns. Access Control is technical (can this user access?); Adalbert Data Use is policy (should this use be permitted?). They compose naturally:

```turtle
# Technical access check (Access Control)
ex:accessRule a odrl:Permission ;
    odrl:action ac:authenticate ;
    odrl:target ex:apiEndpoint .

# Business policy check (Adalbert Data Use)
ex:useRule a odrl:Permission ;
    odrl:assignee ex:analyticsTeam ;
    odrl:action odrl:derive ;
    odrl:target ex:customerData ;
    odrl:constraint [
        a odrl:Constraint ;
        odrl:leftOperand adalbert-due:purpose ;
        odrl:operator odrl:eq ;
        odrl:rightOperand adalbert-due:analytics
    ] .
```

### Big Data Profile

**Focus**: Large-scale data processing rights.

| Aspect | Big Data | Adalbert |
|--------|----------|-------|
| Scale concerns | Yes (batch, stream) | No |
| Processing modes | Yes (aggregate, analyze) | Via DUE profile |
| Formal semantics | No | Yes |

**Integration**: Big Data processing constraints map to ODRL constraints with domain-specific operands:

```turtle
# Big Data processing constraint in Adalbert
ex:rule a odrl:Permission ;
    odrl:constraint [
        a odrl:Constraint ;
        odrl:leftOperand bd:processingMode ;
        odrl:operator odrl:eq ;
        odrl:rightOperand bd:aggregateOnly
    ] .
```

---

## Profile Composition

Adalbert supports clean profile composition through:

### 1. Namespace Separation

The `adalbert:` namespace contains only extensions to ODRL. Domain vocabulary lives in its own namespace:
- `odrl:` -- ODRL 2.2 standard terms (primary)
- `adalbert:` -- Adalbert extensions (State, deadline, DataContract, etc.)
- `adalbert-due:` -- Data use operands and actions
- DCON alignment via `ontology/adalbert-dcon-alignment.ttl`
- `ds:` -- Data sovereignty (external)
- `bd:` -- Big data (external)

### 2. Profile Declaration

Policies declare which profiles they use:

```turtle
ex:policy a adalbert:Subscription ;
    odrl:profile <https://vocabulary.bigbank/adalbert/> ,
                 <https://vocabulary.bigbank/adalbert/due> ,
                 <https://example.org/data-sovereignty> ;
    odrl:assigner ex:dataTeam ;
    odrl:assignee ex:analyticsTeam ;
    # ...
```

### 3. Operand Extension

Domain profiles add operands to ODRL's constraint model. Adalbert adds `adalbert:resolutionPath` for structured resolution:

```turtle
# ODRL constraint using a domain operand with Adalbert resolution
ex:constraint a odrl:Constraint ;
    odrl:leftOperand ds:transferMechanism ;          # From Data Sovereignty
    odrl:operator odrl:eq ;                          # From ODRL
    odrl:rightOperand ds:StandardContractualClauses . # From Data Sovereignty
```

---

## What Adalbert Provides to Other Profiles

### 1. Semantic Precision

Other profiles inherit Adalbert's clarifications:
- Duty lifecycle states (Pending, Active, Fulfilled, Violated)
- Condition vs obligation distinction
- Deterministic evaluation order

### 2. Formal Verification Path

Profiles built on Adalbert can leverage:
- Dafny specifications
- Property-based testing
- Compliance proofs

### 3. Interoperability Foundation

Policies from different domains share:
- Common evaluation semantics
- Consistent conflict resolution (Prohibition > Permission)
- Unified audit trail format

---

## Adoption Scenarios

### Scenario 1: New Domain Profile

A new profile (e.g., "AI Training Data") can:
1. Extend Adalbert (not raw ODRL 2.2)
2. Inherit formal semantics and lifecycle tracking
3. Add domain-specific operands and actions

### Scenario 2: Existing Profile Alignment

An existing profile (e.g., RightsML) can:
1. Create Adalbert-aligned subset
2. Gain deterministic evaluation semantics
3. Retain domain vocabulary

### Scenario 3: Cross-Domain Policy

Organizations using multiple domains can:
1. Compose policies from multiple profiles
2. Evaluate with single Adalbert runtime
3. Maintain unified audit trail

---

## Non-Goals

Adalbert explicitly does **not** attempt to:

1. **Replace domain profiles** -- They add necessary vocabulary
2. **Standardize all domains** -- Each domain knows its needs
3. **Define payment terms** -- Deferred to domain profiles or RL2

---

## References

- [W3C ODRL Profiles Registry](https://www.w3.org/community/odrl/wiki/ODRL_Profiles)
- [ODRL Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
- [Adalbert Data Use Profile](../../profiles/adalbert-due.ttl)
- [Adalbert DCON Alignment](../../ontology/adalbert-dcon-alignment.ttl)
