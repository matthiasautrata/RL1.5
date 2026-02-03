# RL1.5 and the W3C ODRL Profile Ecosystem

**Purpose**: Position RL1.5 within the broader landscape of ODRL 2.2 profiles.

---

## Overview

The W3C ODRL Community Group maintains a [registry of ODRL profiles](https://www.w3.org/community/odrl/wiki/ODRL_Profiles). RL1.5 is designed to complement—not compete with—these domain-specific profiles.

---

## Known ODRL Profiles

| Profile | Domain | Primary Focus | Relationship to RL1.5 |
|---------|--------|---------------|----------------------|
| **Data Sovereignty** | Data governance | Cross-border data flows | Complementary |
| **RightsML** | Media/publishing | News syndication rights | Separate domain |
| **Market Data** | Financial services | Market data licensing | RL1.5 has aligned profile |
| **Big Data** | Data processing | Large-scale data rights | Complementary |
| **Access Control** | Security | Resource authorization | RL1.5 has aligned profile |
| **VC-based** | Identity | Verifiable credentials | Complementary |
| **Language Resources** | NLP/AI | Linguistic data licensing | Separate domain |
| **Regulatory Compliance** | Legal | Policy compliance | Complementary |
| **Privacy Paradigm** | Privacy | Data protection rights | Complementary |
| **ToS Language** | Legal | Terms of service | Separate domain |

---

## RL1.5's Position

### What RL1.5 Is

RL1.5 is a **semantic foundation layer**, not a domain profile. It:

1. **Clarifies ODRL 2.2 semantics** without adding domain terms
2. **Provides formal operational semantics** for verification
3. **Enables domain profiles** to inherit deterministic behavior

### What RL1.5 Is Not

- Not a domain-specific vocabulary
- Not a replacement for existing profiles
- Not an extension of ODRL (it's a subset)

---

## Architectural Relationship

```
                    ┌─────────────────────────────────────┐
                    │         ODRL 2.2 Core               │
                    │    (W3C Recommendation)             │
                    └──────────────┬──────────────────────┘
                                   │
           ┌───────────────────────┼───────────────────────┐
           │                       │                       │
           ▼                       ▼                       ▼
    ┌─────────────┐         ┌─────────────┐         ┌─────────────┐
    │   RL1.5     │         │  RightsML   │         │ Data Sov.   │
    │ (Semantic   │         │  (Domain)   │         │  (Domain)   │
    │  Foundation)│         │             │         │             │
    └──────┬──────┘         └─────────────┘         └─────────────┘
           │
    ┌──────┴──────┬────────────────┐
    │             │                │
    ▼             ▼                ▼
┌────────┐  ┌──────────┐    ┌──────────┐
│ Market │  │ Data Use │    │   DCON   │
│  Data  │  │  Access  │    │ Profile  │
│Profile │  │ Profile  │    │          │
└────────┘  └──────────┘    └──────────┘
```

**Key insight**: RL1.5 sits between ODRL 2.2 Core and domain profiles, providing semantic precision that domain profiles can inherit.

---

## Comparison with Major Profiles

### Data Sovereignty Profile

**Focus**: Cross-border data transfer rules, jurisdiction constraints.

| Aspect | Data Sovereignty | RL1.5 |
|--------|------------------|-------|
| Purpose | Geographic policy | Semantic foundation |
| Adds actions | Yes (transfer, localize) | No |
| Adds operands | Yes (jurisdiction, region) | No (via profiles) |
| Formal semantics | No | Yes |
| Verification target | No | Yes |

**Integration**: Data sovereignty constraints can be expressed as RL1.5 conditions using profile-defined operands:

```turtle
# Data Sovereignty concern expressed in RL1.5
ex:rule rl15:condition [
    rl15:leftOperand ds:dataLocation ;
    rl15:constraintOperator rl15:isAnyOf ;
    rl15:rightOperand ( ds:EU ds:UK ds:CH )
] .
```

### RightsML

**Focus**: News content syndication and media rights.

| Aspect | RightsML | RL1.5 |
|--------|----------|-------|
| Domain | Media/publishing | Cross-domain |
| Vocabulary | Rich (100+ terms) | Minimal |
| Payment modeling | Yes | No (deferred) |
| Formal semantics | No | Yes |

**Integration**: RightsML is domain-specific. RL1.5 would not replace it but could provide underlying semantic clarity if RightsML policies needed formal verification.

### Access Control Profile

**Focus**: Authentication, authorization, resource access.

| Aspect | Access Control | RL1.5 Data Use Profile |
|--------|---------------|------------------------|
| Focus | Technical access | Business authorization |
| Actions | authenticate, authorize | derive, distribute |
| Party model | User, Group, Role | Internal organization |
| Formal semantics | No | Yes |

**Integration**: The profiles address different concerns. Access Control is technical (can this user access?); RL1.5 Data Use is policy (should this use be permitted?). They compose naturally:

```turtle
# Technical access check (Access Control)
ex:accessRule odrl:action ac:authenticate ;
    odrl:target ex:apiEndpoint .

# Business policy check (RL1.5 Data Use)
ex:useRule rl15:action due:derive ;
    rl15:object ex:customerData ;
    rl15:condition [ rl15:leftOperand due:purpose ; ... ] .
```

### Big Data Profile

**Focus**: Large-scale data processing rights.

| Aspect | Big Data | RL1.5 |
|--------|----------|-------|
| Scale concerns | Yes (batch, stream) | No |
| Processing modes | Yes (aggregate, analyze) | Via profiles |
| Formal semantics | No | Yes |

**Integration**: Big Data processing constraints map to RL1.5 conditions:

```turtle
# Big Data processing constraint
ex:rule rl15:condition [
    rl15:leftOperand bd:processingMode ;
    rl15:constraintOperator rl15:eq ;
    rl15:rightOperand bd:aggregateOnly
] .
```

---

## Profile Composition

RL1.5 supports clean profile composition through:

### 1. Namespace Separation

Each profile owns its namespace:
- `rl15:` — RL1.5 core vocabulary
- `rl15-md:` — Market data operands
- `rl15-du:` — Data use operands
- `ds:` — Data sovereignty (external)
- `bd:` — Big data (external)

### 2. Profile Declaration

Policies declare which profiles they use:

```turtle
ex:policy a rl15:Agreement ;
    odrl:profile <https://rl2.org/rl1.5/> ,
                 <https://rl2.org/rl1.5/market-data> ,
                 <https://example.org/data-sovereignty> ;
    # ...
```

### 3. Operand Extension

Domain profiles add operands to RL1.5's core:

```turtle
# RL1.5 core constraint using domain operand
ex:constraint a rl15:AtomicConstraint ;
    rl15:leftOperand ds:transferMechanism ;    # From Data Sovereignty
    rl15:constraintOperator rl15:eq ;          # From RL1.5
    rl15:rightOperand ds:StandardContractualClauses .  # From Data Sovereignty
```

---

## What RL1.5 Provides to Other Profiles

### 1. Semantic Precision

Other profiles inherit RL1.5's clarifications:
- Duty lifecycle states
- Condition vs obligation distinction
- Deterministic evaluation order

### 2. Formal Verification Path

Profiles built on RL1.5 can leverage:
- Dafny specifications
- Property-based testing
- Compliance proofs

### 3. Interoperability Foundation

Policies from different domains share:
- Common evaluation semantics
- Consistent conflict resolution
- Unified audit trail format

---

## Adoption Scenarios

### Scenario 1: New Domain Profile

A new profile (e.g., "AI Training Data") can:
1. Extend RL1.5 (not raw ODRL 2.2)
2. Inherit formal semantics
3. Add domain-specific operands and actions

### Scenario 2: Existing Profile Alignment

An existing profile (e.g., RightsML) can:
1. Create RL1.5-aligned subset
2. Map core constructs to RL1.5 semantics
3. Retain domain vocabulary

### Scenario 3: Cross-Domain Policy

Organizations using multiple domains can:
1. Compose policies from multiple profiles
2. Evaluate with single RL1.5 runtime
3. Maintain unified audit trail

---

## Non-Goals

RL1.5 explicitly does **not** attempt to:

1. **Replace domain profiles** — They add necessary vocabulary
2. **Standardize all domains** — Each domain knows its needs
3. **Add new ODRL features** — It's a subset, not extension
4. **Define payment terms** — Deferred to domain profiles or RL2

---

## References

- [W3C ODRL Profiles Registry](https://www.w3.org/community/odrl/wiki/ODRL_Profiles)
- [ODRL Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
- [RL1.5 Market Data Profile](../profiles/rl1_5-market-data.ttl)
- [RL1.5 Data Use Profile](../profiles/rl1_5-data-use.ttl)
