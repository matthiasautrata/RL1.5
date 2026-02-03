

---

## Appendix A: MDS → RL1.5 Automated Translation Assessment

This appendix assesses the feasibility of automatically translating existing MDS policies to RL1.5, based on approximately **2,000 policies** requiring conversion.

---

### A.1 Translation Feasibility Summary

| Category | Est. % | Translateable | Automation |
|----------|--------|---------------|------------|
| Core permissions | ~70% | ✓ Full | Mechanical |
| Core duties | ~15% | ✓ Full | Mechanical + lifecycle |
| Prohibitions | ~5% | ✓ Full | Mechanical |
| Payment terms | ~5% | ✗ Lossy | Warning + drop |
| Complex constructs | ~5% | ◐ Partial | Manual review |

**Bottom line**: ~85-90% translate mechanically; ~10-15% require warnings or manual review.

---

### A.2 Construct Translation Matrix

#### Fully Translateable (Mechanical)

| MDS Construct | RL1.5 Translation | Transformation |
|---------------|-------------------|----------------|
| `odrl:permission` | `rl15:Privilege` | Type change |
| `odrl:prohibition` | `rl15:Prohibition` | Type change |
| `odrl:duty` | `rl15:Duty` | Type + lifecycle |
| `md:display` | `rl15-md:display` | Namespace swap |
| `md:distribute` | `rl15-md:distribute` | Namespace swap |
| `md:derive` | `rl15-md:derive` | Namespace swap |
| `md:notify` | `rl15-md:notify` | Namespace swap |
| `md:report` | `rl15-md:report` | Namespace swap |
| `md:timeliness` | `rl15-md:timeliness` | Namespace swap |
| `md:assetClass` | `rl15-md:assetClass` | Namespace swap |
| `odrl:constraint` | `rl15:condition` | Restructure |
| `odrl:and` / `odrl:or` | `rl15:And` / `rl15:Or` | Type change |
| All comparison operators | Same | Direct |

#### Translateable with Warnings (Lossy)

| MDS Construct | Translation | Warning |
|---------------|-------------|---------|
| `md:compensate` | Drop | Payment action not expressible |
| `md:invoice` | Drop | Billing action not expressible |
| `md:request` / `md:consent` | Drop | Procedural; handle externally |
| `md:agree` | Drop | Contract formation; use DCON |
| `odrl:remedy` | Drop | Requires RL2 Promise semantics |
| `odrl:consequence` | Drop | Requires RL2 Promise semantics |
| `md:Originator` | → grantor | Supply chain collapsed |
| `md:ServiceFacilitator` | Drop | Intermediary not modeled |
| `md:Controlled` | Drop | DRM semantics not modeled |
| `md:deriveCommingled` | Condition | Subtype → constraint on derive |

#### Translation Failures (Incompatible)

| MDS Construct | Reason | Resolution |
|---------------|--------|------------|
| `odrl:xone` | Not in RL1.5 | Manual rewrite to `or` + mutex |
| Dynamic `AssetCollection` | Non-deterministic | Expand to explicit list |
| Dynamic `PartyCollection` | Non-deterministic | Expand to explicit list |
| Remedy-dependent logic | Semantic dependency | Manual review; may need RL2 |

---

### A.3 Translation Algorithm (Sketch)

```
translate(mds_policy) → (rl15_policy, warnings, errors)

1. Policy structure
   - has parties? → Agreement + grantor/grantee
   - else → Set
   - add profile declarations

2. For each rule:
   - check incompatible constructs → error, skip
   - check lossy constructs → warning, drop or transform
   - map rule type: permission→Privilege, duty→Duty, prohibition→Prohibition
   - map action: namespace swap per table
   - map parties: assignee→subject, target→object
   - restructure constraints → condition tree
   - if Duty: add dutyState=Pending, extract/default deadline

3. Validate output against RL1.5 SHACL

4. Return (policy, warnings, errors)
```

---

### A.4 Effort Estimate (2,000 Policies)

#### Phase 1: Translator Development — 3-4 weeks

| Task | Effort |
|------|--------|
| Core translator (Python/Go) | 2 weeks |
| Namespace mapping tables | 2 days |
| Constraint → condition restructuring | 3 days |
| Duty lifecycle injection | 2 days |
| Warning/error reporting | 2 days |
| SHACL validation integration | 2 days |
| Unit tests | 3 days |

**Deliverable**: `mds-to-rl15` CLI tool

#### Phase 2: Policy Analysis — 1 week

| Task | Effort |
|------|--------|
| Batch translate all 2,000 | 1 day |
| Categorize warnings/errors | 2 days |
| Identify manual review set | 1 day |
| Sample validation | 1 day |

**Deliverable**: Translation report with statistics

#### Phase 3: Translation Execution — 2-3 weeks

| Task | Effort |
|------|--------|
| Automated translation (~1,700-1,800) | 1 day |
| Manual review/rewrite (~200-300) | 1-2 weeks |
| Semantic equivalence testing | 3 days |
| Final SHACL validation | 1 day |

**Deliverable**: 2,000 RL1.5 policies

#### Phase 4: Verification — 1 week

| Task | Effort |
|------|--------|
| Decision equivalence testing | 3 days |
| Edge case review | 2 days |
| Documentation | 1 day |

---

### A.5 Total Effort Summary

| Phase | Duration | Resources |
|-------|----------|-----------|
| Translator development | 3-4 weeks | 1 engineer |
| Policy analysis | 1 week | 1 engineer |
| Translation execution | 2-3 weeks | 1 engineer + 0.5 domain expert |
| Verification | 1 week | 1 engineer |
| **Total** | **7-9 weeks** | **~1.5 FTE average** |

#### Risk Factors

| Risk | Impact | Mitigation |
|------|--------|------------|
| Higher % complex policies | +2 weeks | Early sampling |
| Unknown MDS extensions | +1-2 weeks | Document during analysis |
| Semantic drift | Quality | Extensive decision testing |
| Missing deadline data | Manual effort | Default deadline policy |

---

### A.6 Recommendations

1. **Sample first**: Translate 50-100 policies manually to validate mappings before building tooling.

2. **Preserve originals**: Keep MDS alongside RL1.5 for audit and rollback.

3. **Decision testing**: For each translation, verify same request → same decision (ignoring dropped constructs).

4. **Domain review**: Policies with payment warnings need business sign-off on acceptable loss.

5. **Incremental rollout**: Migrate by policy category, not all at once.

---

### A.7 Success Criteria

| Metric | Target |
|--------|--------|
| Automated translation rate | ≥85% |
| SHACL validation pass | 100% |
| Decision equivalence (sampled) | ≥99% |
| Warnings documented | 100% |
| Manual review completed | 100% of flagged |
