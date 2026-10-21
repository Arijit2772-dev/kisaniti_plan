# Reading guide and source precedence

Prepared 15 October 2026. The current workspace is this folder, not the old absolute paths inside the historical conversation snapshots.

## Reading order

0. [Master plan](MASTER_PLAN.md).
1. Root AGENTS.md and CLAUDE.md.
2. Complete [architecture baseline](architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md), [limited amendments](architecture/2026-10-15_Conversation_Amendments.md) and [blueprint status](blueprint/00_BLUEPRINT_STATUS.md).
3. [Current decision status](blueprint/09_CHAT_DECISION_STATUS.md), [decision register](blueprint/01_DECISION_REGISTER.md), [unknown register](blueprint/02_UNKNOWN_BLOCKER_REGISTER.md), [conflict register](blueprint/04_CONFLICT_REGISTER.md), [rejected/revised positions](blueprint/05_REJECTED_REVISED_DECISIONS.md).
4. [Layer 1–3 integrated handoff](handoff/2026-10-14/KisanNiti_Layers_1_to_3_Handoff.md), [acceptance plan](handoff/2026-10-14/KisanNiti_Layers_1_to_3_Acceptance.md).
5. [Layer 4 v0.2](handoff/2026-10-14/KisanNiti_Layer_4_Review_Proposal.md), [Layer 5 v0.2](communication/KisanNiti_Layer_5_Current.md) and [required WhatsApp/multilingual specification](communication/WHATSAPP_MULTILINGUAL.md).
6. [Quality and verification](QUALITY_AND_VERIFICATION.md), [design alternatives](research/DESIGN_ALTERNATIVES.md), [UI/UX plan](design/UI_UX_PLAN.md), [open issues](OPEN_ISSUES.md), [user actions](USER_ACTIONS.md), [hold-or-sell advisory](design/HOLD_SELL_ADVISORY.md), [privacy contract](design/PRIVACY_CONTRACT.md), [Springer plan](paper/SPRINGER_PUBLICATION_PLAN.md), [Patiala FPO profile](research/PATIALA_FPO_PROFILE_APPROX.md), [Patiala research](research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md), [research and experiment plan](research/RESEARCH_AND_EXPERIMENT_PLAN.md), [paper outline](paper/KisanNiti_Paper_Outline.md).
7. [Hardware](HARDWARE_CURRENT.md), [continuation tasks](blueprint/08_CONTINUATION_TASKS.md), [implementation task template](blueprint/06_IMPLEMENTATION_TASK_TEMPLATE.md).

## What supersedes what

- Explicit later user decisions override the conflicting earlier detail, not unrelated invariants. The amendments document enumerates these changes and their limits.
- Layer 1–3 integrated handoff supersedes the older conversation drafts where it explicitly reconciles them. Layer 4/5 extend it; they do not retroactively prove all proposals accepted.
- Original architecture §§16 and some blueprint examples still name DHT22. Current hardware is SHT30; preserve the original historical file and use the amendment for the new device.
- `KisanNiti_Integration_Testing_and_Hardware_Guide.md` contains superseded Feather/SHT40 purchases. Use its general testing context only; use HARDWARE_CURRENT.md for current hardware.
- The current decision register separates selected design, engineering recommendations, real evidence and unknowns. Never use the newest timestamp alone to promote a proposal to an approved business rule.

## Package organization

This whole folder is **planning only**. Implementation (code, schemas, experiments, evidence, completion reports) lives in [`../../kisaniti_implementation`](../../kisaniti_implementation/README.md); its [CLAUDE.md](../../kisaniti_implementation/CLAUDE.md) explains which changes go where.

- `docs/architecture/`: protected architecture plus explicit dated amendments.
- `docs/blueprint/`: existing governance/unknowns plus current handoff decision status and next tasks.
- `docs/communication/`: current Layer 5 edition and the explicit WhatsApp/multilingual requirement; these supersede the historical snapshot on channel/language scope.
- `docs/handoff/2026-10-14/`: exact copies of all 18 conversation deliverables, including current designs, older drafts, circuit files and documentation-check JSON. Their internal relative links and hashes are preserved.
- `docs/history/repository-before-handoff/`: exact documentation snapshot of the original repository, including its old README/AGENTS/status. It is history, not a second working project.
- `docs/PACKAGE_MANIFEST.json`: copied-file origin/hash records, including the pre-existing untracked constraint catalogue.
- `docs/PACKAGE_VERIFICATION.json`: transfer/link/structure results. These are not application tests.

Earlier absolute paths, dates, source-access probe results and reported checks remain historical context. There is no need to open the old chat folder to read the design. Real providers still require their own current access/terms verification.

The `(3).md` architecture file was byte-identical to the authoritative architecture and omitted as a duplicate. No scratch backend, abandoned thresholds, failed ESP32 toolchain, local credentials or Git internals were copied. No actual trained model or acquired dataset is included because none was established as a completed deliverable.

## Evidence limits

Source materials include cited URLs and prior review findings. Not every URL was reverified during packaging. Some real-source imports were blocked on authorization. The copied JSON checks refer to documentation only; no backend, solver, trained model, database, physical assembly or end-to-end workflow pass is implied.

The existing 21 real-FPO unknown entries remain open. New user context narrows the intended demo setting; it does not validate every FPO's workflow. The imported constraint catalogue was untracked in the original repository; it is supporting material, not evidence of an approved/frozen blueprint phase.
