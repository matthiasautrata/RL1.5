# RL1.5 Documentation

This directory contains comparison and conformance documentation for RL1.5.

---

## Comparison Documents

| Document | Description |
|----------|-------------|
| [comparison-odrl22.md](comparison-odrl22.md) | Detailed comparison with ODRL 2.2 core specification |
| [comparison-odrl-profiles.md](comparison-odrl-profiles.md) | RL1.5 positioning within W3C ODRL profile ecosystem |
| [comparison-w3c-market-data.md](comparison-w3c-market-data.md) | Vocabulary mapping to W3C Market Data ODRL Profile |
| [comparison-dcon.md](comparison-dcon.md) | Alignment analysis with DCON ontology |

## Conformance Documents

| Document | Description |
|----------|-------------|
| [conformance-w3c-best-practices.md](conformance-w3c-best-practices.md) | W3C ODRL Profile Best Practices compliance |
| [namespace-alignment.md](namespace-alignment.md) | Namespace coordination with DCON2 registry |

## Key Findings

### RL1.5 vs ODRL 2.2

- RL1.5 is a **semantic clarification**, not an extension
- Fixes ambiguities: duty lifecycle, condition semantics, evaluation order
- Provides path to formal verification (Dafny, Why3)
- All RL1.5 policies are valid ODRL 2.2 policies

### RL1.5 vs W3C Profiles

- RL1.5 is a **semantic foundation layer**, not a domain profile
- Complementary to domain profiles (RightsML, Data Sovereignty, etc.)
- Domain profiles can build on RL1.5 to inherit formal semantics

### RL1.5 vs W3C Market Data Profile

- RL1.5 market data profile is an **internal governance subset**
- W3C profile models full external vendor supply chain
- Vocabulary mappings preserve interoperability where applicable

### RL1.5 vs DCON

- RL1.5 provides **policy evaluation semantics**
- DCON provides **contract lifecycle management**
- Designed for integration: DCON contracts contain RL1.5 policies
- Promise Theory deferred to RL2 for full alignment

### W3C Conformance

- Follows ODRL Profile Best Practices (August 2024)
- Uses W3C Profiles Vocabulary (DXPROF) for metadata
- SKOS structure for concept organization
- Ready for ODRL Community Group registration

---

## Related Files

| File | Location |
|------|----------|
| DXPROF Profile Declaration | `../ontology/rl1_5-prof.ttl` |
| Core Ontology | `../ontology/rl1_5-core.ttl` |
| SHACL Shapes | `../ontology/rl1_5-shacl.ttl` |
| Market Data Profile | `../profiles/rl1_5-market-data.ttl` |
| Data Use Profile | `../profiles/rl1_5-data-use.ttl` |
| DCON Profile | `../profiles/rl1_5-dcon.ttl` |

---

## External References

- [ODRL 2.2 Information Model](https://www.w3.org/TR/odrl-model/)
- [ODRL 2.2 Vocabulary](https://www.w3.org/TR/odrl-vocab/)
- [W3C ODRL Profiles Registry](https://www.w3.org/community/odrl/wiki/ODRL_Profiles)
- [W3C Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
- [W3C Profiles Vocabulary](https://www.w3.org/TR/dx-prof/)
- [W3C Market Data ODRL Profile](https://www.w3.org/2021/md-odrl-profile/v1/)
- [DCON Ontology](../../dcon/dcon/ontology/dcon.ttl)
