# KisanNiti — project handoff

> **Planning folder only.** This folder describes *how KisanNiti should be* (design, decisions, research plan, paper). All code, contracts-as-files, experiments and test evidence live in the separate implementation folder [`../kisaniti_implementation`](../kisaniti_implementation/README.md) (`/Users/arijitsingh/Documents/kisaniti_implementation`). Do not put code here.

Prepared 15 October 2026 in the user's requested `Documents/kisaniti` folder.

**For Claude or another coding agent: start with [CLAUDE.md](CLAUDE.md), then [the reading guide](docs/START_HERE.md).** Everything needed to reconstruct the documented design is included locally; no access to the previous chat directory is required.

KisanNiti helps an FPO manager choose which selected farmer lots to send to which mandis, on which dates, and in what quantities. Five layers: data collection; cleaning/validation; forecasting and mathematical optimization; manager review/approval; communication, actual sales and remaining-stock feedback.

| Read | Contents |
|---|---|
| [Current decision status](docs/blueprint/09_CHAT_DECISION_STATUS.md) | Agreed choices, recommendations and unresolved evidence |
| [Layer 1–3 handoff](docs/handoff/2026-10-14/KisanNiti_Layers_1_to_3_Handoff.md) | Sources, data volumes, cleaning, models, transport, sensors, optimizer, contracts and work packages |
| [Layer 4](docs/handoff/2026-10-14/KisanNiti_Layer_4_Review_Proposal.md) | Review, edits, approval, reservations, failure handling and integration |
| [Layer 5 — current](docs/communication/KisanNiti_Layer_5_Current.md) | WhatsApp, multilingual farmer instructions, actual sales, reconciliation, evaluation and next plan |
| [Test plan](docs/handoff/2026-10-14/KisanNiti_Layers_1_to_3_Acceptance.md) | Cross-layer acceptance; Layer 4/5 documents extend it |
| [Hardware guide](docs/HARDWARE_CURRENT.md) | Current kit, automatic data path and physical-test limitations |
| [Next tasks](docs/blueprint/08_CONTINUATION_TASKS.md) | How to continue refining contracts before implementation |
| [Open issues](docs/OPEN_ISSUES.md) | 22 issues that can cause later problems, with severity, owner and what "real-world ready" requires |
| [Research and experiment plan](docs/research/RESEARCH_AND_EXPERIMENT_PLAN.md) | EXP-00…14: options compared per layer, metrics, scenarios, decision rules |
| [Paper outline](docs/paper/KisanNiti_Paper_Outline.md) | Research-paper skeleton linked to the experiments; no results yet |
| [Patiala research](docs/research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md) | Storage-risk approximation and likely answers to the 21 unknowns |
| [Blueprint status](docs/blueprint/00_BLUEPRINT_STATUS.md) | Current implementation gate; packaging does not mean implementation is complete |

**Current state:** design documents and circuit files exist. Product implementation, acquired historical dataset, model evaluation, solver benchmarks, physical hardware and end-to-end application tests are not established as complete. Test cases are specified, not executed. Read the gate before coding.

The original repository at `Documents/sem_7/kisaniti` was left unchanged. This folder is the self-contained continuation copy. Historical documents may mention old paths; use this folder and its relative links. No `.git` history, secrets, downloaded runtime, unfinished scratch application or invented dataset has been imported.
