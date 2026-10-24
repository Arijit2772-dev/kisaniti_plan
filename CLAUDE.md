# Claude — start here for KisanNiti

> **Next steps live in [docs/PROJECT_TRACKER.md](docs/PROJECT_TRACKER.md)** (phased checklist, updated 24 Oct 2026).

> **Latest continuation checkpoint (22 Oct 2026):** resume AGMARKNET acquisition from [the Claude continuation handoff](docs/handoff/2026-10-22_Claude_Continuation.md). It lists 11 verified CSV exports still in Downloads pending research import, the exact working report URL, browser recovery notes, and January 2025 as the next missing month. Inspect current files/browser state before resuming; this checkpoint does not change architecture or contract authority.

> **Research paper first (CHAT-27, 21 Oct 2026):** current work happens in [`../kisaniti_research_paper`](../kisaniti_research_paper/RESEARCH_PLAN.md) (research prototypes, experiments, paper). This folder stays the design source of truth.

> **Planning folder only.** This folder describes *how KisanNiti should be* (design, decisions, research plan, paper). Product code, contracts-as-files and product test evidence live in [`../kisaniti_implementation`](../kisaniti_implementation/README.md); research prototypes, experiments and the paper live in [`../kisaniti_research_paper`](../kisaniti_research_paper/README.md) (CHAT-27). Do not put code here.

This folder contains a self-contained project design handoff requested by the user on 14–15 October 2026. The user wants another agent to understand what was decided, improve remaining decisions and proceed toward implementation without hallucinating history.

0. Read the [master plan](docs/MASTER_PLAN.md) first: dual track (CHAT-28), shared `core/`, milestones.
1. Read [AGENTS.md](AGENTS.md) and [docs/START_HERE.md](docs/START_HERE.md).
2. Read the complete [protected architecture](docs/architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md), then [explicit conversation amendments](docs/architecture/2026-10-15_Conversation_Amendments.md).
3. Read [blueprint status](docs/blueprint/00_BLUEPRINT_STATUS.md), [current decision status](docs/blueprint/09_CHAT_DECISION_STATUS.md) and the [unknown register](docs/blueprint/02_UNKNOWN_BLOCKER_REGISTER.md).
4. Read the integrated Layer 1–3, Layer 4 and Layer 5 documents and their acceptance requirements linked in the reading guide. Use earlier layer drafts only for additional history/source details; they do not override the latest integrated design.
5. Read [quality and verification](docs/QUALITY_AND_VERIFICATION.md), [design alternatives](docs/research/DESIGN_ALTERNATIVES.md), [UI/UX plan](docs/design/UI_UX_PLAN.md), [OPEN_ISSUES](docs/OPEN_ISSUES.md), [user actions](docs/USER_ACTIONS.md), [hold-or-sell advisory](docs/design/HOLD_SELL_ADVISORY.md), [privacy contract](docs/design/PRIVACY_CONTRACT.md), [Springer plan](docs/paper/SPRINGER_PUBLICATION_PLAN.md), [Patiala FPO profile](docs/research/PATIALA_FPO_PROFILE_APPROX.md) (what can break later and who decides), the [research and experiment plan](docs/research/RESEARCH_AND_EXPERIMENT_PLAN.md) and the [paper outline](docs/paper/KisanNiti_Paper_Outline.md).
6. Start with [continuation tasks](docs/blueprint/08_CONTINUATION_TASKS.md). State which contract is ready, which is a recommendation and which needs evidence. Do not restart broad architecture brainstorming or pretend product implementation exists.

## Essential context

- College project, practical and inexpensive to assemble; real deployment is an eventual goal, not demonstrated readiness.
- Broad national mandi data is desired; acquire real permitted observations and measure actual usable coverage. Do not inflate counts using fake combinations or upstream model-pretraining totals.
- FPO/member/lot data may be labelled dummy fixtures initially. These are not price-model training labels.
- Current physical context: paddy/wheat bags in farm sheds, electricity and mobile coverage. A Wi-Fi hotspot still must connect the ESP32.
- Latest hardware: ESP32-WROOM-32 + protected I2C SHT30 probe. Automatic readings; no recurring manual moisture-meter step. Hardware can be plugged into the same adapter contract later.
- Storage-condition risk approximation is wanted. Temperature/humidity do not directly measure spoilage, grain moisture or shelf life. Crop-risk policy is still an explicit open task.
- One main ML task: mandi-price forecasting. Cleaning is deterministic initially. Transport uses routes/quotes; FPO quantities are exact records; the optimizer is a constrained mathematical solver. Training is offline and separate from inference/optimization.
- Manager approves exact evaluated revisions and reserves stock. Communication is separate from delivery/consent/sale. Actual outcomes must come from authorized staff or a verified transaction source; sensors cannot infer sales.
- **Required:** live farmer instructions through WhatsApp, in each farmer's recorded preferred language. All members are assumed to have WhatsApp (CHAT-23; the SMS fallback CHAT-21 was withdrawn). Read [current Layer 5](docs/communication/KisanNiti_Layer_5_Current.md) and [WhatsApp/multilingual requirements](docs/communication/WHATSAPP_MULTILINGUAL.md). Punjabi/Hindi/English are the recommended starting languages. Simulated delivery alone is not completion.
- **Demo crop focus (user decision, 15 Oct):** price forecasting and mandi choice are demonstrated on wheat held on the farm and basmati paddy (variety in the series identity). MSP-procured paddy/wheat use a fixed MSP price from a labelled MSP_SCHEDULE source, with no forecast model. Storage-risk approximation: experimental STORAGE-DEMO-0.1. Patiala desk research and DEMO assumptions for all 21 unknowns are in [the research note](docs/research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md); the unknowns stay OPEN.
- Proposed Layer 4/5 demo permissions, fresh-data actions and reconciliation workflows remain recommendations, not proven real-FPO practice.

## First response after reading

Summarize what exists, what has actually been tested, the material open contracts and the next small task. Then continue the authorized design/contract work. Before product coding, explicitly settle the current blueprint gate and task prerequisites. Do not hide that APIs, numeric scales, business policies and real access are not all frozen.

Do not repeatedly ask the user questions already answered in these documents. Ask only for genuinely missing decisions/evidence that cannot be resolved within authorized design scope; continue independent work meanwhile. Explain new ML/model choices and their inputs/evaluation before implementation, as the user requested.
