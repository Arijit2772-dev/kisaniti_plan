# KisanNiti — current five-layer design index

14 October 2026. Start here when continuing the project in another chat or agent.

## Reading order

1. [Layer 1–3 integrated handoff](KisanNiti_Layers_1_to_3_Handoff.md): sources, cleaning, forecasting, storage assessment, transport, optimization, open gates and build order.
2. [Layer 1–3 and cross-layer acceptance plan](KisanNiti_Layers_1_to_3_Acceptance.md): 48 defined scenarios and the distinction between documentation, simulated, real-provider and hardware evidence.
3. [Layer 4 review proposal v0.2](KisanNiti_Layer_4_Review_Proposal.md): manager decisions, atomic approval/reservation, six connected scenarios and 22 acceptance cases.
4. [Layer 5 review proposal v0.1](KisanNiti_Layer_5_Review_Proposal.md): approved communication, actual sales, stock reconciliation, evaluation, full-workflow sequence and 24 acceptance cases.
5. [Current circuit drawing](KisanNiti_Circuit/KisanNiti_Wiring.svg): ESP32-WROOM-32 + protected I2C SHT30 probe. Physical assembly and tests remain separate.

The test lists overlap where they exercise the same boundary. Do not add their counts together and claim that many independent executed tests.

## Authority and current status

All five logical layers now have a written design. This does not mean all exact contracts/business policies are frozen or implementation is complete. Preserve CONFIRMED / DESIGN DECISION / ENGINEERING LEAN / UNKNOWN classifications and distinguish carried-forward decisions from new recommendations.

The repository remains at `/Users/arijitsingh/Documents/sem_7/kisaniti`. Read its AGENTS.md, complete protected architecture, blueprint status and relevant registers before repository work. Promote chat refinements explicitly; do not silently override governance or close real-FPO unknowns.

The current repository status prohibits product coding pending blueprint authorization. This documentation work has not changed that status. Contract freeze and unresolved policy/evidence work remain the next prerequisites; do not treat a proposal or a scratch prototype as accepted implementation.

No application integration, real messaging, physical hardware acceptance, trained forecasting winner or production solver benchmark is established by these documents. Documentation checks only verify document structure, references and illustrative arithmetic.

## Key continuity rules

- Automatic sensor measurement does not automatically observe actual sales. Initial sale recording is a proposed authorized staff workflow; verified transaction integration can be added later.
- A proposal is not approval; approval is not delivery, farmer consent, vehicle booking or sale.
- Linked sales consume matching reservations once; remaining reserved stock is not freely available for a new plan.
- Off-plan reconciliation, amendments, real authority/freshness and storage-risk calibration remain explicit gates.
- Training, inference, optimization and feedback analysis remain separate. No event automatically retrains the model or overwrites approved history.
- Finish the connected workflow and its failure/recovery tests before claiming end-to-end completion.
