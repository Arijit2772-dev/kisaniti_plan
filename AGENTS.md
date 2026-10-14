# KisanNiti — instructions for all agents

> **Planning folder only.** This folder describes *how KisanNiti should be* (design, decisions, research plan, paper). All code, contracts-as-files, experiments and test evidence live in the separate implementation folder [`../kisaniti_implementation`](../kisaniti_implementation/README.md) (`/Users/arijitsingh/Documents/kisaniti_implementation`). Do not put code here.

Read [docs/START_HERE.md](docs/START_HERE.md), the complete protected architecture, the dated conversation amendments, current blueprint status, decision and unknown registers before implementation. Claude-specific entry is [CLAUDE.md](CLAUDE.md); these instructions apply to all agents.

## Authority

- Preserve `CONFIRMED`, `DESIGN DECISION`, `ENGINEERING LEAN`, `UNKNOWN` labels and distinguish acceptance from a proposal merely being discussed.
- The original architecture remains the baseline. Apply only the explicit, limited updates in [conversation amendments](docs/architecture/2026-10-15_Conversation_Amendments.md); unaffected invariants remain unchanged.
- Never infer that the user saying “next layer” approved all numerical settings or real-world assumptions in the preceding proposal.
- The [blueprint status](docs/blueprint/00_BLUEPRINT_STATUS.md) governs product coding. This packaging operation does not silently lift the existing prohibition or freeze APIs/database/policies. Refinement, evidence collection and contract preparation can proceed. When authorized implementation is begun, explicitly reconcile the gate and freeze the affected task contract before dependent code.
- Preserve historical source files. Record refinements in decision/evidence/conflict registers; do not quietly rewrite an UNKNOWN as validated production behavior.

## Work

Use [the task template](docs/blueprint/06_IMPLEMENTATION_TASK_TEMPLATE.md). Work in small scopes with contracts, exclusions, tests and an evidence-based completion report. Continue independent safe work while a source or physical validation is blocked; explain the exact unresolved dependency rather than inventing a substitute.

Preserve tenant isolation, exact inventory/money, historical placement, immutable evaluation evidence, provider/domain separation, distinct planned/approved/actual records, transactional reservations and explicit failure states. Pure optimization and validation make no network calls or database writes.

Simulator and real hardware share a schema but retain origin. No fake real prices, zero fallbacks, automated spoilage claims, unearned model accuracy, or training/integration test completion. Dummy FPOs are explicitly synthetic; no real external messages to fictional contacts.

Do not revive the manual moisture-meter/DHT22/Feather-SHT40 purchase lists, assume every crop/mandi is supported, make GNN compulsory, silently choose transport tariffs or off-plan reservation releases, or equate sensor updates with sales.

Do not send real messages or publish/deploy merely because a test plan describes those actions; use simulated delivery until specifically authorized and configured.

All source and runtime versions, schema changes, assumptions and tests must be recorded. Claims of completion require actual passing evidence for the claimed scope. Documentation checks and mock tests are not real provider, physical device, scientific or full-application validation.
