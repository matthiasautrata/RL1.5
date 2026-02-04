# W3C Market Data ODRL Profile to Adalbert Translation

**Purpose**: The W3C Market Data ODRL Profile (MDS) is an external standard used by data vendors. Vendor agreements expressed in MDS are auto-translated into Adalbert `DataContract` offers and `Set` policies using DUE vocabulary.

---

## How MDS Fits

MDS policies are **external input**, not native Adalbert. The translation pipeline:

```
MDS Policy (vendor) -> translator -> adalbert:DataContract (Offer)
                                  -> odrl:Set (org-wide rules)
```

Internal teams then subscribe to translated contracts (`adalbert:Subscription`) and are additionally governed by org-wide DUE Set policies.

---

## Core Mapping

| MDS Construct | Adalbert Translation | Notes |
|---------------|---------------------|-------|
| `odrl:permission` | `odrl:Permission` | Same type |
| `odrl:prohibition` | `odrl:Prohibition` | Same type |
| `odrl:duty` | `odrl:Duty` | Same type + lifecycle (add `adalbert:state`) |
| `md:display` | `adalbert-due:display` | Namespace swap |
| `md:distribute` | `adalbert-due:distribute` | Namespace swap |
| `md:derive` | `adalbert-due:derive` | Namespace swap |
| `md:timeliness` | `adalbert-due:timeliness` | Namespace swap |
| `md:assetClass` | `adalbert-due:assetClass` | Namespace swap |
| `odrl:constraint` | `odrl:constraint` | Same property |
| Party structure | `odrl:assigner` / `odrl:assignee` | Bilateral agreement |

## Constructs Dropped in Translation

| MDS Construct | Reason |
|---------------|--------|
| `md:compensate`, `md:invoice` | Payment actions -- operational, not policy |
| `md:ServiceFacilitator` | Intermediary role not modeled |
| `odrl:remedy`, `odrl:consequence` | Deferred to RL2 |
| `odrl:xone` | Not supported; manual rewrite to `or` + mutex |

---

## Translation Output

The translator produces:

1. **`adalbert:DataContract`** (subclass of `odrl:Offer`) -- the vendor's terms, with:
   - Provider duties as `odrl:Duty` clauses (via `odrl:obligation`)
   - Consumer permissions as `odrl:Permission` clauses (via `odrl:permission`)
   - Prohibitions as `odrl:Prohibition` clauses (via `odrl:prohibition`)
   - `adalbert:state adalbert:Active` when published

2. **Operand resolution** -- MDS left operands map to DUE operands with `adalbert:resolutionPath`:
   - `md:timeliness` to `adalbert-due:timeliness` (`asset.timeliness`)
   - `md:recipient` to `adalbert-due:role` (`agent.role`)
   - `md:purpose` to `adalbert-due:purpose` (`context.purpose`)

---

## Status

The translator is planned for Phase 2 (implementation). The mapping tables above are validated against the current ontology and DUE profile.
