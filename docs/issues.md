# External Review — Open Issues

Remaining findings from external ODRL/W3C review. All structural and syntactic findings have been resolved (see `decisions-log.md` for the full resolution record). The items below are functional/architectural observations retained for future discussion.

---

### F1 — Authorization and compliance are coupled too tightly

Eval denies on any violated duty (`Adalbert_Semantics.md:674`). A provider SLA miss can deny consumer access. Many data-use teams separate PDP decision from compliance findings.

**Impact:** Operational — affects access decisions in production.

---

### F2 — Duty lifecycle is request-driven, not clock/event-driven

Duties are selected via request matching and updated during Eval. Scheduled duties (delivery/reporting) can be missed if no request arrives.

**Refs:** `Adalbert_Semantics.md:656,663`.

**Impact:** SLA monitoring — missed deadlines go undetected without external triggers.

---

### F3 — Contract dates are modeled but not semantically enforced

`effectiveDate`/`expirationDate` exist in ontology but are not part of PolicyApplicable. An expired subscription can still evaluate as applicable.

**Refs:** `ontology/adalbert-core.ttl:182,188`, `Adalbert_Semantics.md:693`.

**Impact:** Temporal correctness — expired policies can authorize access.

---

### F5 — Recurrence semantics is under-specified for execution

Recurring instance creation is defined abstractly. Lifecycle interaction with policy evaluation cadence is unclear. (Partly addressed by S4 fix to formalize `expand`/`instantiate`/`occurrences`.)

**Ref:** `Adalbert_Semantics.md:430`.

**Impact:** SLA monitoring — real-time monitoring requires clearer instance lifecycle.

---

## Suggested Discussion Priority

1. **F3** — Temporal applicability (expired policies can authorize)
2. **F1** — Auth/compliance coupling (provider miss denies consumer)
3. **F2** — Request-driven duty lifecycle (missed deadlines)
4. **F5** — Recurrence execution cadence (partly addressed by S4)
