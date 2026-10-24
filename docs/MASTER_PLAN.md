# KisanNiti master plan — publishable paper + working software + working hardware

21 October 2026 • The single plan that ties everything together. The user's goals:

1. A **Springer paper** that is good enough to publish: real data, real model runs, fair comparisons, every claim and reference verified.
2. **Software** that works directly with no errors and can demonstrate every layer.
3. **Hardware** that works and can be shown live.
4. **Everything documented**, layer by layer.

Research and implementation run **side by side** (CHAT-28).

## 1. Definition of done

| Deliverable | Done means |
|---|---|
| **Paper 1** (Springer conference, LNNS/LNCS style) | Every number comes from a script-produced result. Every reference is opened and checked. Comparisons use baselines and significance tests. Only finished, tested parts are described; everything else is Future Work. Guide-reviewed, humanized, plagiarism-checked, in the venue template, declarations complete |
| *(no second paper — CHAT-33)* | Everything is in the one paper; what the data cannot support is a limitation and future work |
| **Software** | The full flow runs on one command (Docker Compose): data → cleaning → forecast with ranges → storage risk → hold-or-sell → fair plan → manager approval → WhatsApp to a test phone → sale from J-form → stock update → next plan. All acceptance tests in CI pass. Mobile-friendly UI in Punjabi/Hindi/English. DEMO label wherever data is synthetic |
| **Hardware** | A real ESP32 + SHT30 node in a grain container sending readings into the live app, calibrated, outage-tested, plus a Wokwi copy anyone can run |
| **Documentation** | Per layer: how it works, why this choice vs alternatives, how to run, tests, results. A user manual, deployment guide, demo script and paper reproducibility guide |

## 2. Three repos, one shared core (the key design for "verified once, used twice")

| Repo | Holds |
|---|---|
| `kisaniti` (plan) | Design, decisions, this plan |
| `kisaniti_implementation` | The product. It contains **`core/`**, a pure Python package with every algorithm: cleaning rules, forecasting evaluation, storage rule, hold-or-sell, optimizer, validator and exact quantity math. It also holds the API (FastAPI), database (PostgreSQL), web app (Next.js), WhatsApp adapter, MQTT ingestion and firmware |
| `kisaniti_research_paper` | Experiments, results, figures and paper text. Experiments **import the same `core/` package** (installed from the implementation repo), so the algorithm in the paper is exactly the algorithm in the product |

**Why:** if research and product each had their own code, they would drift apart, and the paper could describe something the software does not do. With one tested core, any fix improves both. Research-only code (model training scripts, plotting) stays in the research repo.

## 3. Decisions taken for this plan

All are recorded in the [decision register](blueprint/01_DECISION_REGISTER.md).

| ID | Decision |
|---|---|
| CHAT-28 | Dual track. The product gate opens **layer by layer**: a layer's product code may start once its contracts are frozen (W01 slice for that layer) and recorded |
| CHAT-29 | Hardware: **ESP32-WROOM-32 + protected SHT30 probe in the grain**, built for real, plus a Wokwi "digital twin" for reproducibility. The v2 DHT22 + DS18B20 circuit is retired (history only) |
| CHAT-30 | One frontend: the **product Next.js app** supplies the paper's screenshots. No separate Streamlit prototype |
| CHAT-31 | Manuscript in **LaTeX with the Springer template**, unless the chosen venue or the guide requires Word |

## 4. The layer-by-layer sequence

Every layer goes through the same eight steps, so nothing is skipped:

1. **Freeze contract:** exact fields and failure states for this layer (planning repo).
2. **Core algorithm** in `core/`, with unit, property and reference tests.
3. **Research experiment** using the core on real data: comparisons and results (research repo).
4. **Product integration:** API, database, UI screen, background jobs.
5. **Acceptance tests** from the planning documents, run in CI on real PostgreSQL.
6. **Docs:** layer guide (how it works, why, how to run, limits).
7. **Paper section** drafted from the results.
8. **Review:** self-check against the [quality plan](QUALITY_AND_VERIFICATION.md), plus `/code-review` for money, stock, optimizer and privacy code.

| Milestone | Layer / work | Needs | Exit criteria |
|---|---|---|---|
| **M0** Foundation (week 1) | Contract freeze for shared basics (IDs, tenants, exact kg/₹, dates, provenance, errors); `core/` skeleton; FastAPI + PostgreSQL + Next.js skeleton; Docker Compose; GitHub Actions CI; research env pinned; v2 results reproduced | — | CI green; one health page loads; v2 numbers reproduced or differences explained |
| **M1** Literature (weeks 1–2, parallel) | Verified review, comparison matrix, sourced cost parameters | — | All references verified; Section 2 drafted |
| **M2** L1 Data (weeks 1–2) | Loaders, manifests, coverage report for paddy, wheat and basmati; product import job and data screen | **User downloads** wheat/basmati (CAPTCHA) | Manifests and coverage table; data visible in the app |
| **M3** L2 Cleaning (week 2) | Rules + comparison on a hand-checked sample; readiness reports in the app | M2 | Counts reconcile; Section 3.2 |
| **M4** L3a Forecasting (weeks 2–4) | Full model comparison with ranges; promoted model per crop/horizon served by the API; price screen with ranges | M3 | DM tests, coverage checks; Sections 3.3/4.1 |
| **M5** L3b Storage + hardware (weeks 1–4, parallel) | Wokwi twin; real node built and calibrated; MQTT ingestion; storage rule; scenario experiment; device and lot screens | Kit purchase | Real readings visible live; calibration and outage tests recorded; Sections 3.4/4.2 |
| **M6** L3c Hold-or-sell (weeks 4–5) | Rule in core; backtest; advice screen | M4, M5 | Backtest table; Sections 3.5/4.3 |
| **M7** L3d Allocation + fairness (weeks 1–4, parallel) | Exact lexicographic MILP, oracle, validator, solver comparison, price of fairness; plan generation in the app | M0 | 100% oracle agreement; scaling results; Sections 3.6/4.4 |
| **M8** L4 Manager review (weeks 4–5) | Proposal list/detail/edit/approve/release with atomic reservations | M7 | L4 acceptance tests incl. concurrency pass |
| **M9** L5 WhatsApp + outcomes (weeks 5–6) | Templates in 3 languages, Meta test number, delivery status, J-form sale entry, reconciliation | M8; **user's WhatsApp test number** | Real message on test phone; L5/WA tests pass |
| **M10** Integration (weeks 6–7) | End-to-end run, whole-system backtest, demo script, failure drills | All | One-command demo works from a clean machine; Section 4.6 |
| **M11** Paper finish (weeks 7–8) | Assemble v3; numbers script; reference check; humanizer; guide review; template; declarations | All | Submission package ready |
| **M12** Release (week 8) | Tag versions in all repos; archive code/data scripts (e.g. Zenodo DOI); user manual and demo video | M10–M11 | Tagged release; docs complete |

Timeline assumes data access in week 1. **Every week of data delay moves M3–M6 and the paper by a week.**

## 5. Hardware plan (best option for paper + live demo)

**Why ESP32 + SHT30:**
- cheap;
- a digital I²C sensor that is more accurate than DHT22 (compare datasheets);
- a protected probe that can sit inside the grain;
- in-grain humidity approximates water activity, which links directly to published mould limits.

It needs no manual moisture readings, and the same design works for wheat and basmati. Optional future add-ons: a CO₂ sensor for early spoilage signs, and a second probe at a different depth.

| Step | Output |
|---|---|
| H1 Buy kit ([list](HARDWARE_CURRENT.md)), wire per circuit, check probe labels | Photo of the build |
| H2 Firmware (pinned Arduino/PlatformIO): device ID, boot ID, sequence number, 5-min readings, buffer, MQTT over TLS | Firmware v1 in the implementation repo |
| H3 Wokwi twin with the same firmware (SHT3x part or custom chip) | Public Wokwi link for reviewers |
| H4 Salt calibration (~75% RH) and settling-time test in grain | Offset and settling time recorded |
| H5 Outage tests: Wi-Fi off, power off, broker off, reboot | Data-loss and recovery report |
| H6 One-week run in a labelled wheat container (dry, then humidified) | Real sensor plot + storage categories for the paper |
| H7 One-time moisture-meter check at a mandi | Grounds the risk rule |
| H8 Demo setup: node + phone hotspot + laptop showing the live app | Ready to show |

## 6. Verification at every level

- **Data:**
  - raw files kept unchanged with checksums;
  - row reconciliation;
  - 50-row manual check against the website for every crop;
  - synthetic data always labelled.
- **Code:**
  - CI on every push: lint, type check, unit, property and contract tests;
  - real PostgreSQL integration tests;
  - nightly end-to-end runs;
  - optimizer checked against brute force.
- **Research:**
  - time-ordered splits, held-out last season;
  - baselines and significance tests;
  - seeds and versions logged;
  - negative results reported;
  - paper numbers read from results by script.
- **References:** every DOI opened, every claim checked, logged in `verification_log.md`.
- **Writing:** plain human prose; humanizer pass; guide review; college plagiarism check.
- **Demo:** run the demo script from a fresh clone on a second machine before showing it.

Full rules: [quality and verification](QUALITY_AND_VERIFICATION.md).

## 7. Documentation set (produced as we go)

| Document | Where |
|---|---|
| Layer guides L1–L5 (how it works, why vs alternatives, run, tests, results) | `kisaniti_implementation/docs/layers/` |
| API reference (auto-generated) and data contracts | implementation repo |
| User manual for FPO staff (with screenshots, 3 languages) | implementation repo |
| Deployment and demo guide (one command, troubleshooting) | implementation repo |
| Hardware build guide (photos, wiring, calibration) | implementation repo `firmware/` |
| Experiment reports and reproducibility guide | research repo |
| Paper + declarations | research repo `paper/` |
| Decisions, unknowns, design | this planning repo |

## 8. How the work is split between sessions

Run separate sessions in parallel, with **one owner per folder** so they never edit the same files:

- **Session A — Literature (M1):** research repo `literature/`, paper Section 2.
- **Session B — Foundation (M0), then Allocation (M7):** implementation `core/` skeleton, CI, Docker, and research env.
- **Session C — Hardware (M5):** implementation `firmware/`, research `hardware/` and the L3b experiments.
- **After data arrives:** Session D — Data, Cleaning, Forecasting (M2–M4).
- **Later:** Sessions E — Review and WhatsApp (M8–M9); F — Integration and paper (M10–M11).

Each session updates the STATUS files and pushes; this main session coordinates and checks their results.

## 9. Risks and how we handle them

| Risk | Plan |
|---|---|
| Wheat/basmati data not obtained | **Critical path.** The user downloads with a CAPTCHA in the browser (Claude can drive the in-app browser while the user solves it), or gets CEDA access. If impossible, Paper 1 focuses on paddy, allocation, fairness and IoT, and says so |
| Models don't beat simple baselines | Report it; it is still a finding (v2 already shows this for MSP crops) |
| Optimizer too slow at 1,000 farmers | Progressive leximin variant; report realistic sizes |
| Hardware delays | The Wokwi twin keeps the pipeline testable; real-node results are added only if collected in time |
| WhatsApp test number setup issues | Simulated adapter for development; the live screenshot needs only one working test message |
| Over-claiming | Paper describes only finished, tested parts; the checklist in M11 enforces it |
| Unverified references | Session A verification log; nothing goes in without a checked DOI |

## 10. What only the user does (in priority order)

Details in [USER_ACTIONS](USER_ACTIONS.md):

1. **Download wheat and basmati AGMARKNET data** (or get CEDA access). This is the most urgent item.
2. Buy the hardware kit.
3. Create a Meta WhatsApp test number.
4. Confirm the venue with the guide; send author emails and the corresponding author.
5. Ask the guide about ethics for any human testing.
6. Later: hosting choice and when to make the repos public.
