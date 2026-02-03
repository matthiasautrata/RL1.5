# RL1.5 Profiles

Domain-specific vocabularies built on RL1.5 Core.

---

## Profile Architecture

```
                    rl1_5-core.ttl
                    (Norm, Condition, Policy)
                           │
                           ▼
              rl1_5-governance-core.ttl
              (Shared operands: purpose, classification,
               jurisdiction, retention, audit)
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
         ▼                 ▼                 ▼
  rl1_5-market-data   rl1_5-data-use    rl1_5-dcon
  (display, seats,    (role, dept,      (Promise ext,
   redistribution)     environment)      contract status)
```

---

## Profile Descriptions

### rl1_5-governance-core.ttl

**Shared vocabulary for all data governance domains.**

| Category | Operands |
|----------|----------|
| Purpose | `purpose` (with hierarchy) |
| Classification | `classification`, `sensitivity` |
| Ownership | `dataOwner`, `dataSubject`, `dataSteward` |
| Jurisdiction | `jurisdiction`, `residency` |
| Temporal | `retentionPeriod`, `effectiveWindow`, `expiry` |
| Processing | `processingMode` |
| Audit | `auditRequired`, `loggingRequired` |

**Actions**: `use`, `read`, `distribute`, `delete`, `modify`, `aggregate`, `anonymize`, `derive`, `log`, `notify`

---

### rl1_5-market-data.ttl

**Vocabulary for market data licensing contracts.**

| Category | Operands |
|----------|----------|
| Display | `displayType` (display, nonDisplay) |
| Redistribution | `redistributionAllowed`, `redistributionType` |
| Derived Data | `derivedDataRestrictions` |
| Entitlements | `entitlementTier`, `concurrentSeats`, `deviceLimit`, `namedUsers` |
| Timeliness | `delayedData`, `delayMinutes`, `realTimeAccess` |
| Metering | `usageMetering`, `queryCount`, `dataVolume` |
| Audit | `auditRight`, `reportingInterval` |

**Actions**: `displayData`, `processData`, `redistribute`, `createDerived`, `reportUsage`, `permitAudit`

**Use cases**:
- Bloomberg/Reuters license enforcement
- Display vs non-display access control
- Seat-based entitlement management
- Derived data compliance

---

### rl1_5-data-use.ttl

**Vocabulary for internal data access control.**

| Category | Operands |
|----------|----------|
| Roles | `role` (with hierarchy) |
| Organization | `department`, `businessUnit`, `costCenter`, `memberOf` |
| Environment | `environment` (prod/dev/staging/sandbox), `region` |
| Approval | `approvalRequired`, `approvalStatus`, `legalReview` |
| Break-Glass | `breakGlass`, `breakGlassJustification` |
| Separation | `separationOfDuty`, `previousActor` |
| Lineage | `dataLineage`, `thirdPartyTransfer`, `internalUseOnly` |

**Actions**: `access`, `query`, `export`, `share`, `requestApproval`, `grantApproval`, `revokeAccess`, `reviewAccess`

**Use cases**:
- Role-based access control (RBAC)
- Purpose limitation enforcement
- Break-glass emergency access
- Separation of duties

---

### rl1_5-dcon.ttl

**Conformance declaration for DCON (Data Contract Ontology).**

Establishes that DCON is an RL1.5 profile with a Promise extension.

| DCON Concept | RL1.5 Mapping |
|--------------|---------------|
| `dcon:DataContract` | `rl15:Set` (offer) |
| `dcon:DataContractSubscription` | `rl15:Agreement` (bilateral) |
| `odrl:permission` | `rl15:Privilege` |
| `odrl:duty` | `rl15:Duty` |
| `dcon:Promise` | Profile extension (not RL2) |

**Key distinction**: DCON Promise is commitment tracking. RL2 Promise is full Promise Theory with automatic remediation.

---

## Using Profiles

### Import in Turtle

```turtle
@prefix rl15mkt: <https://rl1-5.example/profiles/market-data#> .

# Declare profile usage
ex:myPolicy a rl15:Agreement ;
    odrl:profile <https://rl1-5.example/profiles/market-data> ;
    rl15:clause [
        a rl15:Privilege ;
        rl15:subject ex:TradingDesk ;
        rl15:action rl15mkt:displayData ;
        rl15:object ex:BloombergFeed ;
        rl15:condition [
            a rl15:AtomicConstraint ;
            rl15:leftOperand rl15mkt:displayType ;
            rl15:constraintOperator rl15:eq ;
            rl15:rightOperand rl15mkt:display
        ]
    ] .
```

### Combining Profiles

Profiles can be combined:

```turtle
ex:complexPolicy a rl15:Agreement ;
    odrl:profile <https://rl1-5.example/profiles/market-data> ,
                 <https://rl1-5.example/profiles/data-use> ;
    # Can use operands from both profiles
    ...
```

---

## Creating New Profiles

1. **Import governance core**:
   ```turtle
   owl:imports <https://rl1-5.example/profiles/governance> .
   ```

2. **Define domain-specific operands**:
   ```turtle
   ex:myOperand a rl15:LeftOperand ;
       rl15gov:contextKey "myValue" ;
       rl15gov:supportedOperators (rl15:eq rl15:isAnyOf) .
   ```

3. **Define domain-specific actions**:
   ```turtle
   ex:myAction a rl15:Action ;
       rl15:includedIn rl15gov:use .
   ```

4. **Document resolution semantics** in prose (for implementation)

---

## Validation

Use SHACL to validate policies against profiles:

```bash
# Validate against RL1.5 core
pyshacl -s ontology/rl1_5-shacl.ttl -d my-policy.ttl

# Validate against specific profile
pyshacl -s profiles/rl1_5-market-data-shacl.ttl -d my-policy.ttl
```

---

## Profile Extension Rules

Profiles MAY:
- Add new `rl15:LeftOperand` instances
- Add new `rl15:Action` instances with `includedIn` hierarchies
- Add new `skos:Concept` values for operands
- Add domain-specific classes that don't conflict with RL1.5

Profiles MUST NOT:
- Use RL2-exclusive concepts (Claim, Power, Liability, Immunity)
- Override RL1.5 Core semantics
- Define new norm types (only Privilege, Duty, Prohibition)
- Define new policy types (only Set, Agreement)

---

## Roadmap

| Profile | Status | Next Steps |
|---------|--------|------------|
| governance-core | Draft 0.1 | Review operand completeness |
| market-data | Draft 0.1 | Add exchange-specific constraints |
| data-use | Draft 0.1 | Add ABAC patterns |
| dcon | Draft 0.1 | Verify DCON 1.0 compatibility |

---

*Profiles extend RL1.5 with domain vocabulary while preserving formal semantics.*
