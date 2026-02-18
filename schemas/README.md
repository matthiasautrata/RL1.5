# Adalbert Schemas — LinkML/YAML Parity

YAML/LinkML representation of the Adalbert ODRL 2.2 profile.
Semantically equivalent to the hand-written Turtle files in `ontology/` and `profiles/`.

---

## Directory Structure

```
schemas/
├── adalbert-odrl.yaml         # ODRL 2.2 base class/slot wrappers (schema)
├── adalbert-core.yaml         # Adalbert extensions: contracts, states, slots (schema)
├── adalbert-due.yaml          # DUE vocabulary schema: classes + container slots only
├── vocabulary/
│   ├── operands.yaml          # 35 DUE LeftOperand instances (34 DUE + 1 ODRL re-annotated)
│   ├── actions.yaml           # 13 DUE Action instances (ODRL common actions used directly)
│   └── concepts.yaml          # 44 SKOS Concept right-operand values
├── examples/
│   ├── data-contract.yaml     # Bilateral contract + subscription (no blank nodes)
│   ├── data-use-policy.yaml   # Internal access policy with role/purpose constraints
│   └── dprod/                 # 8 dprod-contracts translations
│       ├── basic-data-contract.yaml
│       ├── provider-promises.yaml
│       ├── consumer-promises.yaml
│       ├── full-lifecycle.yaml
│       ├── api-sla.yaml
│       ├── data-quality.yaml
│       ├── support-promise.yaml
│       └── marketplace-offer.yaml
└── README.md                  # this file
```

---

## Design Principles

### Not the Author-Friendly Dialect

The `linkml/` directory (at repo root) contains an author-friendly authoring dialect — simplified slot names, bilateral structure, enum-based validation — for data stewards writing contracts. **These `schemas/` files are different.** They target ontology-level parity with the RDF files in `ontology/` and `profiles/`.

### Semantic Equivalence, Not Structural Identity

The YAML files are semantically equivalent to the Turtle files. They are not structurally identical. Key difference: **no blank nodes**. All duties, permissions, prohibitions, and constraints receive explicit IDs. This is an improvement over the Turtle representation — better for registries, event sourcing, and incremental updates.

Blank node example (Turtle):
```turtle
ex:contract odrl:obligation [
    a odrl:Duty ;
    adalbert:subject ex:team ;
    odrl:action adalbert-due:deliver ;
] .
```

Named node equivalent (YAML/generated RDF):
```yaml
obligation:
  - id: "ex:contract/duties/delivery"
    subject: "ex:team"
    action: "adalbert_due:deliver"
```

Generated RDF:
```turtle
ex:contract odrl:obligation ex:contract/duties/delivery .
ex:contract/duties/delivery a odrl:Duty ;
    adalbert:subject ex:team ;
    odrl:action adalbert-due:deliver .
```

Semantically equivalent. ODRL processors handle both forms identically.

### Manual Synchronization

Changes to `ontology/` or `profiles/` TTL files must be manually reflected in `schemas/`. Generation-from-LinkML (using `gen-owl` or `gen-rdf`) is under evaluation but not yet the primary workflow.

---

## Schema Import Chain

```
adalbert-odrl.yaml
    └── adalbert-core.yaml
            └── adalbert-due.yaml
                    └── vocabulary/operands.yaml
                    └── vocabulary/actions.yaml
                    └── vocabulary/concepts.yaml
```

Example files import `adalbert-core.yaml`.

---

## Vocabulary Coverage

| File | Count | Source |
|------|-------|--------|
| `vocabulary/operands.yaml` | 35 operands | `profiles/adalbert-due.ttl` operand declarations |
| `vocabulary/actions.yaml` | 13 DUE actions | `profiles/adalbert-due.ttl` action declarations |
| `vocabulary/concepts.yaml` | 44 concepts | `profiles/adalbert-due.ttl` concept declarations |

ODRL Common Vocabulary actions (use, read, display, distribute, delete, modify, aggregate, anonymize, derive, inform, reproduce) are not redeclared — they are used directly by URI.

---

## Gap Register

Known deviations between the LinkML representation and the OWL/TTL representation.
Documented here as accepted gaps, not defects.

| Gap | RDF Construct | LinkML Status | Mitigation |
|-----|--------------|---------------|------------|
| Transitivity | `owl:TransitiveProperty` on `adalbert:partOf` and `adalbert:memberOf` | Not expressible | Documented in slot annotation; runtime engine enforces during target/party resolution |
| Disjointness | `owl:disjointWith` on `adalbert:Subscription` / `adalbert:DataContract` | Not expressible | Documented; SHACL-only (see `ontology/adalbert-shacl.ttl`) |
| Union domains | `rdfs:domain [ owl:unionOf (...) ]` on `adalbert:state`, `adalbert:effectiveDate`, `adalbert:expirationDate` | Not expressible (`domain:` is single class in LinkML) | Accepted; schema documents the intended domain in description |
| subPropertyOf generation | `rdfs:subPropertyOf` on `adalbert:subject` (→ `odrl:assignee`), `adalbert:object` (→ `odrl:function`), `adalbert:partOf` (→ `odrl:partOf`), `adalbert:memberOf` (→ `odrl:partOf`) | `is_a` on slots may generate `rdfs:subPropertyOf` in `gen-owl` — requires verification | Documented in slot annotations with commented `is_a:` |
| Dual typing | `a odrl:LeftOperand, skos:Concept` on DUE operands; `a odrl:Action, skos:Concept` on DUE actions | Not directly expressible | Custom generator step or post-processing needed to emit both `rdf:type` triples |
| owl:oneOf on State | `owl:oneOf (adalbert:Pending ...)` | LinkML `enum:` with `meaning:` URIs | Functionally equivalent for authoring; OWL-level closed enumeration axiom not generated |
| Complex SHACL | `sh:SPARQLTarget`, `sh:targetNode` enumeration, 11 rejection shapes | Not expressible | Stays in `ontology/adalbert-shacl.ttl` |
| SHACL rejection shapes | 11 shapes rejecting unused ODRL features | Not expressible | Stays in `ontology/adalbert-shacl.ttl` |
| Action hierarchy inference | `odrl:includedIn` as subsumption | `includedIn` slot on Action instances | Custom reasoner; not OWL inference |
| LogicalConstraint semantics | `odrl:and`, `odrl:or` strict boolean evaluation | Expressible structurally; semantics not enforced | Runtime validator enforces |
| `adalbert:not` | Logical negation on `LogicalConstraint` | `not` slot on `Adalbert_LogicalConstraint` | Structural expression only; runtime enforces semantics |
| Profile declaration | W3C DXPROF (`prof:isProfileOf`, `prof:hasResource`) | Not a standard LinkML concept | Expressed in `ontology/adalbert-prof.ttl`; not mirrored in schemas/ |
| SKOS ConceptScheme | `adalbert-due:scheme a skos:ConceptScheme` | `SKOSConceptScheme` class in `adalbert-due.yaml`; instance in `concepts.yaml` | Structural parity; fully expressed |

**Accepted gaps (by design):**
- OWL axioms (transitivity, disjointness, union domains) — reasoning-layer concern, not authoring
- Complex SHACL rejection shapes — RDF-only; not replicated
- `odrl:includedIn` subsumption inference — stays in ODRL reasoning engine
- Profile declaration resource descriptors — stays in `adalbert-prof.ttl`

---

## Coverage Assessment

| Dimension | Coverage |
|-----------|----------|
| Classes and hierarchy | ~95% |
| Properties/slots | ~90% |
| SKOS vocabulary | ~100% |
| DUE operand instances | 100% |
| DUE action instances | 100% (13 DUE-specific; ODRL common used directly) |
| SKOS concept instances | 100% |
| SHACL basic constraints (cardinality, datatype) | Generated by `gen-shacl` |
| SHACL complex shapes | Not expressed (stays in adalbert-shacl.ttl) |
| Instance examples (no blank nodes) | 100% |

---

## Using the Schemas

### Validation (LinkML)

Instance files in `examples/` and `vocabulary/` include file-format metadata fields
(`schema`, `prefixes`, `type`, `parties`, `assets`) that are human-readable annotations,
not schema slots. These fields are outside strict schema validation.

To validate an instance, use `--target-class` with a clean (metadata-free) YAML object:

```bash
# Validate a DataContract instance
linkml-validate --schema schemas/adalbert-core.yaml --target-class DataContract <contract.yaml>

# Validate a Subscription instance
linkml-validate --schema schemas/adalbert-core.yaml --target-class Subscription <subscription.yaml>

# Validate an individual Duty
linkml-validate --schema schemas/adalbert-core.yaml --target-class Adalbert_Duty <duty.yaml>

# Validate a DUE LeftOperand instance
linkml-validate --schema schemas/adalbert-due.yaml --target-class DUELeftOperand <operand.yaml>

# Validate a DUE Action instance
linkml-validate --schema schemas/adalbert-due.yaml --target-class DUEAction <action.yaml>

# Validate a SKOS Concept
linkml-validate --schema schemas/adalbert-due.yaml --target-class SKOSConcept <concept.yaml>
```

**Clean DataContract instance (strips metadata headers):**
```yaml
# Minimal valid DataContract for linkml-validate
id: "ex:my-contract"
label: "My Contract"
profile:
  - "https://vocabulary.bigbank/adalbert/"
assigner: "ex:provider"
target: "ex:my-dataset"
state: Active
effectiveDate: "2026-01-01T00:00:00Z"
obligation:
  - id: "ex:my-contract/duties/delivery"
    label: "Daily delivery"
    subject: "ex:provider"
    action: "adalbert_due:deliver"
    recurrence: "FREQ=DAILY;BYHOUR=6;BYMINUTE=0"
    deadline: "PT30M"
permission:
  - id: "ex:my-contract/permissions/use"
    label: "Use access"
    action: "odrl:use"
```

### Generation (LinkML generators)

```bash
# Generate OWL from the core schema
gen-owl schemas/adalbert-core.yaml

# Generate SHACL from the core schema
gen-shacl schemas/adalbert-core.yaml

# Generate Python Pydantic models
gen-pydantic schemas/adalbert-core.yaml

# Generate JSON Schema
gen-json-schema schemas/adalbert-core.yaml
```

---

## Relationship to Other Files

| This file | Mirrors |
|-----------|---------|
| `adalbert-odrl.yaml` | ODRL 2.2 base vocabulary (W3C) |
| `adalbert-core.yaml` | `ontology/adalbert-core.ttl` |
| `adalbert-due.yaml` | `profiles/adalbert-due.ttl` (schema level only) |
| `vocabulary/operands.yaml` | `profiles/adalbert-due.ttl` operand declarations |
| `vocabulary/actions.yaml` | `profiles/adalbert-due.ttl` action declarations |
| `vocabulary/concepts.yaml` | `profiles/adalbert-due.ttl` concept declarations |
| `examples/data-contract.yaml` | `examples/data-contract.ttl` |
| `examples/data-use-policy.yaml` | `examples/data-use-policy.ttl` |
| `examples/dprod/*.yaml` | `examples/dprod-translated.ttl` |
