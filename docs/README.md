# RL1.5 Documentation

---

## Specification

| Document | Description |
|----------|-------------|
| [RL1_5_Semantics.md](RL1_5_Semantics.md) | Formal semantics (normative) |

## Conformance

| Document | Description |
|----------|-------------|
| [conformance-w3c-best-practices.md](conformance-w3c-best-practices.md) | W3C ODRL Profile Best Practices compliance |

## Comparisons

| Document | Description |
|----------|-------------|
| [comparison-odrl22.md](comparisons/comparison-odrl22.md) | vs ODRL 2.2 core |
| [comparison-odrl-profiles.md](comparisons/comparison-odrl-profiles.md) | vs W3C ODRL profile ecosystem |
| [comparison-w3c-market-data.md](comparisons/comparison-w3c-market-data.md) | vs W3C Market Data Profile (+ migration appendix) |
| [comparison-dcon.md](comparisons/comparison-dcon.md) | vs DCON ontology |
| [namespace-alignment.md](comparisons/namespace-alignment.md) | DCON2 namespace coordination |

---

## Key Findings

### RL1.5 vs ODRL 2.2

- Semantic clarification, not extension
- Fixes: duty lifecycle, condition semantics, evaluation order
- Path to formal verification
- All RL1.5 policies are valid ODRL 2.2

### RL1.5 vs W3C Market Data

- RL1.5 models internal governance
- W3C models external vendor supply chain
- ~85-90% policies translate mechanically (see Appendix A)

### RL1.5 vs DCON

- RL1.5: policy evaluation semantics
- DCON: contract lifecycle management
- Designed for integration

---

## Related Files

| File | Location |
|------|----------|
| Core Ontology | `../ontology/rl1_5-core.ttl` |
| SHACL Shapes | `../ontology/rl1_5-shacl.ttl` |
| DXPROF Declaration | `../ontology/rl1_5-prof.ttl` |
| Domain Profiles | `../profiles/` |

---

## External References

- [ODRL 2.2 Information Model](https://www.w3.org/TR/odrl-model/)
- [W3C Profile Best Practices](https://www.w3.org/community/reports/odrl/CG-FINAL-profile-bp-20240808.html)
- [W3C Market Data Profile](https://www.w3.org/2021/md-odrl-profile/v1/)
