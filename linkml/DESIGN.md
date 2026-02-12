# Adalbert LinkML Metamodel — Design

**Goal**: A LinkML schema that defines a convenient YAML authoring format for Adalbert data contracts and policies. The YAML is the primary artifact. Translation to RDF, JSON-LD, SHACL, Postgres, etc. happens downstream — either via LinkML generators or custom transformers.

**Non-goals**: Faithful OWL mirror. Nested complexity. Semantipolis dependency.

---

## Design Principles

1. **Author-first, not ontology-first.** Slot names are what a data steward would write (`provider`, `target`, `deadline`), not what ODRL calls them (`odrl:assigner`, `odrl:target`, `adalbert:deadline`). The `slot_uri` annotation carries the semantic mapping invisibly.

2. **Bilateral structure is positional.** Provider obligations and consumer obligations are separate YAML lists. The `subject`/`object` roles are derived from list membership + the contract's provider/consumer fields. No one writes `adalbert:subject` in YAML.

3. **Single constraint per rule.** Composed constraints (AND/OR/NOT) are out of scope for the authoring format. An AI-assisted UI or a power-user escape hatch can handle those later. A single `constraint:` block with three fields covers 90%+ of real contracts.

4. **Enums are the vocabulary.** Actions, operands, operators, and concept values are LinkML enums with `meaning:` URIs. Authors write `deliver`, not `adalbert-due:deliver`. The schema resolves.

5. **Flat, not nested.** No class hierarchies deeper than one level. No recursive types. Lists of simple objects.

6. **Two top-level document types.** A file is either a DataContract or a Subscription. One root class per file type.

7. **Target inheritance is a convention, not a mechanism.** The translator/validator enforces that rules without an explicit `target` inherit from the policy. The schema just makes `target` optional on rules.

---

## Schema Files

```
linkml/
├── DESIGN.md                  ← this document
├── adalbert.yaml              ← core structural schema (classes, slots)
└── adalbert_vocab.yaml        ← vocabulary enums (actions, operands, values)
```

`adalbert.yaml` imports `adalbert_vocab.yaml`. Both import `linkml:types`.

No dependency on Semantipolis `core.yaml`. We define a minimal local base (just enough for identifier + description mixins) or inline the few slots we need.

---

## Class Inventory

### DataContract (tree_root)

A provider's offer. The primary authoring target.

| Slot | Type | Required | Maps to | Notes |
|------|------|----------|---------|-------|
| `id` | string | yes (identifier) | — | Short name, e.g. `market-prices-v2` |
| `label` | string | no | `rdfs:label` | Human-readable title |
| `description` | string | no | `dcterms:description` | |
| `provider` | string | yes | `odrl:assigner` | Party short name or URI |
| `target` | string[] | yes | `odrl:target` | Asset(s) covered |
| `state` | State | no | `adalbert:state` | Default: Active |
| `effective_date` | datetime | no | `adalbert:effectiveDate` | |
| `expiration_date` | datetime | no | `adalbert:expirationDate` | |
| `previous_version` | string | no | `prov:wasRevisionOf` | ID of prior contract |
| `provider_obligations` | Obligation[] | no | `odrl:obligation` | Subject = provider (implicit) |
| `consumer_obligations` | Obligation[] | no | `odrl:obligation` | Subject = consumer (deferred to subscription) |
| `permissions` | Permission[] | no | `odrl:permission` | |
| `prohibitions` | Prohibition[] | no | `odrl:prohibition` | |

### Subscription

An activated contract binding both parties.

| Slot | Type | Required | Maps to | Notes |
|------|------|----------|---------|-------|
| `id` | string | yes (identifier) | — | |
| `label` | string | no | `rdfs:label` | |
| `contract` | string | yes | `adalbert:subscribesTo` | DataContract ID |
| `provider` | string | yes | `odrl:assigner` | |
| `consumer` | string | yes | `odrl:assignee` | |
| `target` | string[] | yes | `odrl:target` | |
| `state` | State | no | `adalbert:state` | |
| `effective_date` | datetime | yes | `adalbert:effectiveDate` | |
| `expiration_date` | datetime | no | `adalbert:expirationDate` | |
| `provider_obligations` | Obligation[] | no | `odrl:obligation` | |
| `consumer_obligations` | Obligation[] | no | `odrl:obligation` | |
| `permissions` | Permission[] | no | `odrl:permission` | |
| `prohibitions` | Prohibition[] | no | `odrl:prohibition` | |

### Obligation

A duty owed by one party. Which party is determined by which list it appears in.

| Slot | Type | Required | Maps to | Notes |
|------|------|----------|---------|-------|
| `label` | string | no | `rdfs:label` | |
| `action` | Action | yes | `odrl:action` | From Action enum |
| `target` | string | no | `odrl:target` | Override — if absent, inherited from contract |
| `recurrence` | string | no | `adalbert:recurrence` | RFC 5545 RRULE |
| `deadline` | string | no | `adalbert:deadline` | ISO duration (`PT30M`) or datetime |
| `constraint` | Constraint | no | — | Single constraint (inlined) |

### Permission

A usage right granted to consumers.

| Slot | Type | Required | Maps to | Notes |
|------|------|----------|---------|-------|
| `label` | string | no | `rdfs:label` | |
| `action` | Action | yes | `odrl:action` | |
| `target` | string | no | `odrl:target` | Override |
| `assignee` | string | no | `odrl:assignee` | Explicit in Subscription; absent in Offer |
| `constraint` | Constraint | no | — | |

### Prohibition

A restriction on consumers.

| Slot | Type | Required | Maps to | Notes |
|------|------|----------|---------|-------|
| `label` | string | no | `rdfs:label` | |
| `action` | Action | yes | `odrl:action` | |
| `target` | string | no | `odrl:target` | Override |
| `assignee` | string | no | `odrl:assignee` | |
| `constraint` | Constraint | no | — | |

### Constraint

An atomic condition on a rule.

| Slot | Type | Required | Maps to | Notes |
|------|------|----------|---------|-------|
| `left` | Operand | yes | `odrl:leftOperand` | From Operand enum |
| `operator` | Operator | yes | `odrl:operator` | From Operator enum |
| `right` | string | yes | `odrl:rightOperand` | Value — string, enum ref, or literal |

---

## Enum Inventory

### State

`Pending`, `Active`, `Fulfilled`, `Violated` — maps to `adalbert:Pending` etc.

### Operator

`eq`, `neq`, `gt`, `gteq`, `lt`, `lteq` — maps to `odrl:eq` etc.

Also: `isPartOf`, `isAllOf`, `isAnyOf`, `isNoneOf` for set-based operators.

### Action

Combined ODRL Common Vocabulary + DUE actions. One flat enum.

**ODRL standard** (meaning → `odrl:` URI):
`use`, `read`, `display`, `distribute`, `delete`, `modify`, `aggregate`, `anonymize`, `derive`, `inform`, `reproduce`

**DUE-specific** (meaning → `adalbert-due:` URI):
`nonDisplay`, `deliver`, `notify`, `report`, `conformTo`, `log`, `export`, `copy`, `link`, `profile`, `query`, `calculateIndex`, `algorithmicTrading`

### Operand

All DUE left operands. One flat enum.

`purpose`, `classification`, `sensitivity`, `assetClass`, `market`, `isBenchmark`, `jurisdiction`, `residency`, `retentionPeriod`, `expiry`, `processingMode`, `auditRequired`, `role`, `organization`, `costCenter`, `project`, `recipientType`, `environment`, `network`, `timeliness`, `delayMinutes`, `legalBasis`, `consentId`, `accessPattern`, `volumeLimit`, `rateLimit`, `derivationType`

Each has a `meaning:` pointing to the `adalbert-due:` or `odrl:` URI, and an annotation for `resolutionPath`.

### Concept Value Enums (grouped by domain)

These provide valid values for the `right` field when the `left` operand expects a controlled vocabulary term.

| Enum | Values | Used with operand |
|------|--------|-------------------|
| `Purpose` | analytics, research, compliance, operations | `purpose` |
| `Classification` | public, internal, confidential, restricted | `classification` |
| `Sensitivity` | pii, mnpi, phi | `sensitivity` |
| `Environment` | production, staging, development, sandbox | `environment` |
| `Network` | internalNetwork, externalNetwork, cloudNetwork | `network` |
| `Timeliness` | realtime, nearRealtime, delayed, endOfDay, historical | `timeliness` |
| `ProcessingMode` | human, automated, modelTraining, inference | `processingMode` |
| `LegalBasis` | consent, contract, legalObligation, vitalInterest, publicTask, legitimateInterest | `legalBasis` |
| `AccessPattern` | batch, streaming, interactive, api | `accessPattern` |
| `DerivationType` | commingled, nonSubstitutive, newProduct | `derivationType` |
| `RecipientType` | internal, professional, retail | `recipientType` |

The `right` field on Constraint is typed as `string`, not as a union of these enums. This keeps the schema simple and extensible — custom values are just strings. Validation that `right` matches the expected vocabulary for a given `left` operand is a second-pass concern (SHACL, custom validator, or UI logic).

---

## Simplifications and Tradeoffs

| Simplification | What's lost | Why it's OK |
|----------------|-------------|-------------|
| No LogicalConstraint (AND/OR/NOT) | Complex constraint composition | Single constraints cover 90%+. AI UI handles the rest. |
| No inline blank nodes | RDF structural purity | Everything gets a name. Better for registries anyway. |
| `right` is string, not typed | Static type-checking of operand values | Keeps schema simple. Validation is downstream. |
| No Party/Asset as first-class objects | Can't define parties/assets inline | They're referenced by short name. A separate registry (or just convention) resolves them. |
| Provider/consumer derived from list position | Can't have mixed-party obligation lists | Bilateral structure is the whole point. Mixing would be confusing. |
| No `object` (counterparty) on duties | Can't specify who gets notified | Derivable: provider duties → object is consumer, and vice versa. Add `object` slot later if needed. |

---

## Example Instance: DataContract

```yaml
id: market-prices-v2
label: Market Price Data Contract v2
provider: data-platform-team
target:
  - market-prices
state: Active
effective_date: "2026-01-01T00:00:00Z"
previous_version: market-prices-v1

provider_obligations:
  - label: Daily delivery
    action: deliver
    recurrence: "FREQ=DAILY;BYHOUR=6;BYMINUTE=0"
    deadline: PT30M

  - label: Schema change notification
    action: notify
    target: schema-changes
    deadline: P14D

consumer_obligations:
  - label: Monthly usage reporting
    action: report
    target: usage-stats
    deadline: P30D

permissions:
  - action: display

  - action: nonDisplay
    constraint:
      left: recipientType
      operator: eq
      right: internal

  - action: derive

prohibitions:
  - action: distribute
```

## Example Instance: Subscription

```yaml
id: quant-market-data-2026
label: Quant Research Market Data 2026
contract: market-prices-v2
provider: data-platform-team
consumer: quant-research
target:
  - market-prices
effective_date: "2026-01-15T00:00:00Z"
expiration_date: "2026-12-31T23:59:59Z"

provider_obligations:
  - label: Daily delivery
    action: deliver
    recurrence: "FREQ=DAILY;BYHOUR=6;BYMINUTE=0"
    deadline: PT30M

  - label: Schema change notification
    action: notify
    target: schema-changes
    deadline: P14D

consumer_obligations:
  - label: Monthly usage reporting
    action: report
    target: usage-stats
    deadline: P30D

permissions:
  - action: display
    assignee: quant-research

  - action: nonDisplay
    assignee: quant-research

  - action: derive
    assignee: quant-research

prohibitions:
  - action: distribute
    assignee: quant-research
```

## Example Instance: Simple Read-Only Policy

```yaml
id: ref-data-readonly
label: Reference Data Read-Only
provider: data-platform-team
target:
  - reference-data

permissions:
  - action: read

prohibitions:
  - action: modify
```

---

## What LinkML Generators Produce

From the schema + instance data:

| Generator | Output | Use |
|-----------|--------|-----|
| `gen-json-schema` | JSON Schema | Form validation in UI, API validation |
| `gen-json-ld-context` | JSON-LD context | Semantic interop, RDF conversion |
| `gen-shacl` | SHACL shapes | RDF graph validation |
| `gen-pydantic` | Python models | Programmatic use, NiceGUI form binding |
| `gen-markdown` | Docs | Human reference |
| `gen-sqltables` | SQL DDL | Contract registry in Postgres |
| `linkml-validate` | Validation report | Instance validation against schema |

The `gen-pydantic` output is particularly useful for the NiceGUI app — it gives typed Python classes that can drive form generation and validation directly.

---

## Decisions

1. **Scope: Contracts only.** DataContract and Subscription. Data Use Policies (`odrl:Set`) deferred.

2. **Namespace resolution: separate config.** Instance YAML uses short names. A separate config file maps them to URIs. Not a schema concern.

3. **No `object` slot for now.** Counterparty on duties is derivable (provider duties → object is consumer, vice versa). Add the optional slot later when the simplification proves insufficient.

4. **Multi-document files: separate files.** One root per file, linked by `contract:` reference.

5. **Schema references in obligations.** Cross-validation is a toolchain concern. The `target` field is just a string.

6. **Schema versioning.** Tracks Adalbert version (currently 0.7).

---

## Implementation Sequence

1. Write `adalbert_vocab.yaml` — enums only, mechanical transcription from `adalbert-due.ttl`
2. Write `adalbert.yaml` — classes + slots, importing vocab
3. Port the three `.ttl` examples to instance YAML
4. Run `linkml-validate` on the instances
5. Run `gen-json-schema`, `gen-pydantic`, verify outputs
6. Use `gen-pydantic` output to prototype NiceGUI form
