# Next work for Claude or another agent

15 October 2026. Continue toward implementation without restarting the five-layer design. Do not interpret packaging as proof all contracts are frozen. The current blueprint status still prohibits product coding until explicitly advanced; the original source repository is untouched.

## First task: complete the executable contract blueprint

**Scope:** review current decisions/amendments and prepare versioned exact contracts/examples. No broad application implementation or new ML architecture.

**Inputs:** complete architecture, current five-layer handoffs, unknown register, existing proposed C01–C15/L4C01–06/L5C01–08 contracts and test requirements.

**Outputs:** a small set of explicit schema/policy decisions plus producer/consumer examples and recorded unresolved dependencies. Use the standard task template for each implementable slice.

**Start with:** canonical identities/tenant boundaries; exact quantity/money/rate serialization and precision/ranges; UTC/business-date semantics; source provenance/real-synthetic labels; sensor event identity/time-quality/assignment; inventory snapshot and error/readiness contracts. Choose engineering details explicitly with rationale rather than asking the user to repeat settled context.

**Crop scope for the demo (CHAT-20):** fixtures and forecasting scenarios use held wheat and basmati paddy (e.g. 1121/1509 as distinct series). Add an MSP_SCHEDULE contract (commodity, grade, season, procurement window, ₹/quintal, notification source/version) for MSP-procured paddy/wheat. These follow the fixed-price path and are excluded from forecast training/evaluation. Use STORAGE-DEMO-0.1 and the Part C DEMO assumptions from the [Patiala research note](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md) as starting values, each labelled DEMO.

**Then freeze the necessary DEMO policy bundle:** permissions, increments, allowed calendar dates, applicability, transport fixture semantics, storage approximation status, approval freshness, solver candidate eligibility, sale finalization and off-plan/correction behavior. Unvalidated LIVE counterparts stay gated. Explain new model inputs/evaluation before implementation.

**Tests:** valid/invalid/missing/duplicate/correction/time/tenant examples, precise producer-consumer expectations and links to the existing acceptance scenarios. A schema is not complete until the consumer can distinguish every required failure state.

**Exclusions:** no fabricated historical dataset, forecast winner, crop thresholds, real-FPO workflow, customer permissions or successful hardware/model/runtime tests. No live messages, purchase or deployment merely to demonstrate a design.

**Exit:** affected contracts are explicitly frozen with decisions/evidence; current blueprint status and coding authorization have been reconciled before the first product-code task. Do not bypass the status by implementing an entire product under a “prototype” label.

## Where work happens

Design, decision and policy work: this planning folder. Code, machine-readable contracts, experiments, test evidence and completion reports: [`kisaniti_implementation`](../../../kisaniti_implementation/README.md), tracked in its [STATUS.md](../../../kisaniti_implementation/STATUS.md).

## Prerequisites only the user can provide

See [OPEN_ISSUES](../OPEN_ISSUES.md): authorise the coding gate (OI-10); register for market-data access on data.gov.in/CEDA (OI-05); approve Git setup (OI-09); one conversation with a Patiala FPO CEO and an arhtiya (OI-16); college ethics guidance (OI-20); hardware purchase and bench tests. Agents continue independent work meanwhile and must not invent these.

## Research track

Run experiments EXP-00…14 from the [research plan](../research/RESEARCH_AND_EXPERIMENT_PLAN.md) alongside the work packages (EXP-01 right after data access; EXP-08 after the optimizer oracle). Results feed the [paper outline](../paper/KisanNiti_Paper_Outline.md). New design work from OI-01/02/06 (hold/sell advisory, required intervals, privacy contract) belongs in the contract-freeze task.

## Subsequent small work packages

1. Tenant-aware exact values, configuration, audit and migration foundation after contract freeze.
2. Seeded organization/location fixtures and tested exact inventory boundary.
3. Bounded authorized market import plus source manifests; report access blockers while fixture work proceeds.
4. Cleaning/readiness and reproducible dataset release; no invented observations or future leakage.
5. Sensor simulator, registered ingestion and UNKNOWN assessment path; physical adapter later.
6. Baseline forecasting and actual held-out evaluation; candidate comparison/promotion only from evidence.
7. Route/quote adapters with explicit applicability; transport math verified independently.
8. Tiny optimizer oracle, sparse input, exact reconstruction and full objective-sequence verification; then solver benchmarking.
9. Planning snapshots and immutable proposals; integrated Layer 1→2→3 scenario.
10. Layer 4 actual-database approval/reservation, duplicate/concurrent requests and durable handoff.
11. Layer 5 simulated delivery, linked sales, release and new planning cycle; gated reconciliation/correction.
12. Physical ESP32 integration and real providers only when available/authorized; separate evidence from mocks.
13. Full failure/recovery/backup and end-to-end acceptance; report the exact tested scope and remaining pilot gaps.

Use the more detailed W00–W12 plan in the integrated handoff as the dependency reference. Entire archive acquisition and GNN research are not prerequisites for a small correctly labelled connected demonstration. Do not introduce unrelated infrastructure.

## Completion report for every task

Record changed files, agreed contract/policy versions, actual test procedures/results, data/fixture origin, limitations, deviations and next task. Unknown/blocked/skipped is not passed. A model fixture is not a trained model; a simulated message is not delivery; an approved plan is not a sale.

## Required communication refinement

Read [current Layer 5](../communication/KisanNiti_Layer_5_Current.md) and [WhatsApp/multilingual requirements](../communication/WHATSAPP_MULTILINGUAL.md). Freeze farmer contact/language/permission, template version/locale mapping and webhook contracts. Implement/test the WhatsApp adapter and approved launch-language variants; simulation is a development stage, not a scope reduction. No new translation ML model is required.
