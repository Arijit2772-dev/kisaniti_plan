# KisanNiti Repository Instructions

These instructions apply repository-wide.

## Before any work

1. Read the complete architecture source of truth: [`docs/architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md`](docs/architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md).
2. Read [`docs/blueprint/00_BLUEPRINT_STATUS.md`](docs/blueprint/00_BLUEPRINT_STATUS.md).
3. Check the relevant [`decision register`](docs/blueprint/01_DECISION_REGISTER.md) and [`UNKNOWN`/blocker register`](docs/blueprint/02_UNKNOWN_BLOCKER_REGISTER.md) entries before implementation.
4. If the blueprint status prohibits implementation, do not create or change product code.

## Non-negotiable governance

- Preserve the architecture classifications: `CONFIRMED`, `DESIGN DECISION`, `ENGINEERING LEAN`, and `UNKNOWN`.
- Never turn an `UNKNOWN` into implementation behavior without an explicit approved decision supported by appropriate evidence.
- Never silently revive an idea marked rejected, revised, refined, corrected, deferred, or unknown.
- Stop and record/report conflicts rather than guessing or silently changing architecture.
- Do not introduce unnecessary infrastructure, dependencies, or technology choices.

## Architectural invariants to preserve

- Multi-FPO isolation.
- Exact inventory arithmetic.
- Auditability and explicit failure semantics.
- Provider/domain separation.
- Planned information remains separate from actual reality.
- Raw/authoritative facts, derived information, system decisions, human decisions, and reality remain separate.

## Requirements for every implementation task

Use [`docs/blueprint/06_IMPLEMENTATION_TASK_TEMPLATE.md`](docs/blueprint/06_IMPLEMENTATION_TASK_TEMPLATE.md). Each task must be small, contract-driven, and auditable, with at least:

- scope;
- contract;
- explicit exclusions;
- tests;
- completion report.

If required behavior is missing, contradictory, or still `UNKNOWN`, stop and report the blocker. Do not fill the gap with plausible behavior.
