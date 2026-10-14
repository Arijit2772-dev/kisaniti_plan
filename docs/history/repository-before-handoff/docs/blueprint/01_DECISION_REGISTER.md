# Decision Register

This register records future explicit decisions without replacing the [architecture source of truth](../architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md). It contains no newly invented product or architectural decisions at Phase 0.

Existing architecture decisions remain indexed by their headings and classifications in the source. A future entry must preserve the architecture labels exactly: `CONFIRMED`, `DESIGN DECISION`, `ENGINEERING LEAN`, or `UNKNOWN`.

Do not use this register to resolve an `UNKNOWN` without approved evidence, to revive a revised/rejected idea, or to silently override the architecture. Record conflicts first in [`04_CONFLICT_REGISTER.md`](04_CONFLICT_REGISTER.md).

## Decision entry template

```text
Decision ID:
Area:
Classification:
Status:

Question:

Decision:

Why this decision is needed:

Architecture constraints:

Evidence:

Alternatives considered:

Why selected:

Why alternatives were rejected/deferred:

Failure modes:

Debugging implications:

Testing implications:

Security implications:

Performance implications:

Extensibility implications:

Affected modules/contracts:

Open questions:

Revisit trigger:

Sources:

Date:
```

## Entry acceptance checks

- The question and decision are explicit and small enough to audit.
- Classification and status are not conflated.
- Evidence is recorded in [`03_EVIDENCE_REGISTER.md`](03_EVIDENCE_REGISTER.md) where applicable.
- Affected `UNKNOWN` entries are linked but not deleted; their resolution and status are updated audibly.
- Conflicts and superseded positions are preserved rather than overwritten.
