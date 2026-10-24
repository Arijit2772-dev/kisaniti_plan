# KisanNiti project tracker — phases and steps

24 October 2026 • The single checklist for what to do next. Tick `[x]` when done and add the date and the evidence link. Details of each milestone are in the [master plan](MASTER_PLAN.md); this file is the order of work. **Who:** `AI` = an agent can do it; `YOU` = only the project owner can; `BOTH` = the owner enters a CAPTCHA, signs off or supplies a fact while an agent works.

How to use it: finish the steps of a phase in order; steps in different phases marked ∥ can run at the same time in separate sessions (one session per folder).

## 0. Where things stand (24 Oct 2026)

| Area | State | Evidence |
|---|---|---|
| Planning (architecture, 5 layers, decisions CHAT-01…32) | Done | this repo |
| Data | **13 basmati months** (Sep 2024–Jan 2025, Sep 2025–Feb 2026, Sep–Oct 2026) + Paddy(Common) + 4 wheat months filed with checksums; **17 months still to download** | research `layers/L1_data/download_log.csv` |
| Literature review R1 | Done (38 studies, 57 verified references); authors must check | research `literature/` |
| v2 reproduction R0 | Done: 222 of 222 numbers match | research `results/R0_v2_reproduction` |
| Storage rule + simulation (L3b) | Done in simulation; real node needs the kit | research `results/L3b_storage`, product `core.storage` (31 tests) |
| Allocation core (L3d) | Core + oracle + validator built, CI green; the comparison experiments are unfinished | product `core/`, research `layers/L3d_allocation_fairness` |
| Firmware + Wokwi twin | Firmware builds; MQTT leg of the twin not yet observed | product `firmware/` |
| Product skeleton (API, DB, Next.js, Compose, CI) | Exists, CI green on the last 5 pushes; no manager screens yet | product repo |
| Paper | v2 exists (13–14 pp.); v3 LaTeX base + figure drafts; editorial revision **not yet in the repo** (see Phase H) | research `paper/` |
| Hardware | Not purchased | — |

**Reconciliation notes (found when reviewing the work done by the user and a second AI agent on 22 Oct):**

- The other agent's package of revised v2 files was kept in its own temporary folder and is **not on this machine**. Only its decisions survive, and Phase H redoes the work.
- Several STATUS files still say "not started" for work that exists; Phase 0 fixes them.
- The folders `~/Documents/kisaniti_final`, `untitled folder` and `untitled folder 2` are not part of the project (two are empty; `untitled folder` holds copies of the same CSV files and an unrelated PDF).
- `~/Documents/downloads` holds the original 12 CSV exports. They are filed and checksum-verified; keep them as a backup.

## Phase 0 — Housekeeping (do first, about one session)

- [x] 0.1 Review everything the user and the other agent did (24 Oct)
- [x] 0.2 File the 12 basmati CSVs with manifests and exact provenance; checksums match the hand-over receipt (24 Oct) — `AI`
- [x] 0.3 Apply the editorial decisions (passive voice, author order, `[1][4]` brackets) to the writing guide (24 Oct) — `AI`
- [x] 0.4 Commit the other agent's handoff notes and the editorial decision in this repo (24 Oct) — `AI`
- [x] 0.5 Fix stale STATUS rows in the research and product repos (R0, R2, R7, W01) (24 Oct) — `AI`
- [ ] 0.6 Decide what to do with the empty/duplicate folders in `~/Documents` (delete or keep) — `YOU`

## Phase A — Data completion (L1) — critical path

Why first: forecasting, the hold-or-sell backtest and the whole-system result all depend on enough basmati seasons.

- [ ] A.1 Download the **remaining 17 months** of Paddy(Basmati), Punjab — `BOTH` (you type each CAPTCHA; an agent fills the form and files it using the [runbook](../../kisaniti_research_paper/layers/L1_data/DOWNLOAD_RUNBOOK.md)):
  - [ ] February 2025 (the only gap in the 2024–25 season)
  - [ ] 2023-24 season: Sep, Oct, Nov, Dec 2023, Jan 2024
  - [ ] 2022-23 season: Sep 2022 – Jan 2023
  - [ ] 2021-22 season: Sep 2021 – Jan 2022
  - [ ] One off-season test month (June 2025) to confirm basmati is quiet outside the season
  - Note: Feb 2026 had only 3 rows and Jan 2026 only 27, so older seasons need Sep–Jan only; the site offers 2021 onward.
- [ ] A.2 Write the coverage report (rows, mandis, gaps and price spread per month and variety) and decide the "forecastable series" rule (EXP-01) — `AI`
- [ ] A.3 Collect the official **MSP values** for paddy and wheat for 2021-22 to 2026-27 with source links, for the MSP_SCHEDULE — `AI` ∥
- [ ] A.4 Optional: apply for CEDA access for pre-2021 history; only needed if three seasons are not enough — `YOU`
- [ ] A.5 Register on data.gov.in for the daily API (needed for the live demo, not for the paper) — `YOU` ∥

Done when: ≥ 4 full basmati seasons are filed, the coverage table exists, and the MSP table is sourced.

## Phase B — Cleaning (L2, EXP-02)

- [ ] B.1 Build the deterministic cleaner: units, dates, min ≤ modal ≤ max, duplicates, variety-name mapping (1121/1509/…), and a rule for **blank-date rows** (Nov 2024 has 108, Dec 2024 has 50) — `AI`
- [ ] B.2 Draw a stratified sample (~300 rows) and **hand-check about 50 of them against the AGMARKNET website**; the checker is recorded — `YOU` (an agent prepares the sheet)
- [ ] B.3 Compare cleaning options (rules only, +MAD flag, +Hampel, +IQR) by precision/recall and by effect on forecast error — `AI`
- [ ] B.4 Dataset release with manifest and row reconciliation (raw = kept + removed + flagged) — `AI`

Done when: counts reconcile exactly and the chosen rule set is justified by the comparison.

## Phase C — Forecasting (L3a, EXP-03/04) — needs A and B

- [ ] C.1 Freeze the forecast contract slice (ranges required beyond 7 days, CHAT-22) — `AI`
- [ ] C.2 Baselines (naive, seasonal naive, moving average) and the MSP rule per variety — `AI`
- [ ] C.3 ETS/ARIMA, then XGBoost/LightGBM with lags, calendar and arrivals — `AI`
- [ ] C.4 GRU, T-GCN, A3T-GCN (reuse the v2 code) and Chronos-2 zero-shot — `AI`
- [ ] C.5 Prediction ranges: quantile models, conformal intervals, Chronos quantiles; coverage check — `AI`
- [ ] C.6 Rolling-origin evaluation, held-out last season, Diebold–Mariano tests, per-variety results and figures — `AI`
- [ ] C.7 Promotion rule applied: the winner per variety/horizon, or "baseline is enough" — `AI`

Done when: results files and figures exist from scripts and negative results are reported.

## Phase D — Hold-or-sell (L3c, EXP-07) — needs C

- [ ] D.1 Hold-or-sell rule in the shared `core/` with tests (exact decimals, the seven decision branches) — `AI`
- [ ] D.2 Parameter table: storage loss, holding cost, interest — sourced values from the literature review; otherwise labelled assumptions with ranges — `AI`
- [ ] D.3 Backtest on the basmati seasons against: sell at harvest, hold to a fixed month, median-forecast rule, perfect foresight — `AI`
- [ ] D.4 Sensitivity analysis and scenario seasons — `AI`

## Phase E — Allocation and fairness (L3d, EXP-08/09) ∥ can run now

The core, oracle and validator exist; the experiments were interrupted by the usage limit.

- [ ] E.1 Finish the policy comparison on the 300-member Patiala persona (nearest mandi, independent choice, weighted sum, max-min, utilitarian, leximin, perfect foresight) with real prices — `AI`
- [ ] E.2 Solver comparison and scaling grid (HiGHS, SCIP, CP-SAT, greedy) with timings — `AI`
- [ ] E.3 Price of fairness and sensitivity (transport rate, daily limits) — `AI`
- [ ] E.4 Re-run with basmati prices once Phase A has them — `AI`
- [ ] E.5 Figures and paper sections 3.6 / 4.4 — `AI`

## Phase F — Storage and hardware (L3b) ∥

- [x] F.1 Storage rule, firmware, Wokwi twin and simulated scenarios (22 Oct)
- [ ] F.2 Observe the Wokwi MQTT leg end to end and record a screenshot — `BOTH`
- [ ] F.3 **Buy the kit** (ESP32-WROOM-32, protected SHT30 probe, resistors, cables, USB adapter, about 1 kg grain) — `YOU`
- [ ] F.4 Wire and flash it; run the salt calibration, settling-time and outage tests (the [build guide](../../kisaniti_implementation/firmware/BUILD_GUIDE.md)) — `YOU`, an agent writes up the results
- [ ] F.5 One-week run in a labelled grain container (dry, then humidified) — `YOU`
- [ ] F.6 One-time moisture-meter check at a mandi — `YOU`

Paper 1 can go out with the Wokwi simulation only; the real results strengthen Paper 2.

## Phase G — Product build (layer by layer, product repo)

Each layer follows the eight steps of the master plan (freeze contract → core → experiment → API/UI → tests → docs → paper section → review).

- [ ] G.1 Shared basics, L3b ingestion and L3d allocation slices frozen (done); freeze the **L1, L2, L3a, L3c, L4, L5** slices — `AI`
- [ ] G.2 L1/L2: import job, data and cleaning screens — `AI`
- [ ] G.3 L3a/L3c: price and advice screens with ranges — `AI`
- [ ] G.4 L3d: plan generation and the proposal screens — `AI`
- [ ] G.5 L4: review, edit, approve/reserve, release; concurrency tests on real PostgreSQL — `AI`
- [ ] G.6 L5: WhatsApp templates in 3 languages, test message to your own phone, J-form sale entry — `BOTH` (needs the Meta test number)
- [ ] G.7 UI polish with your design ideas — `BOTH`
- [ ] G.8 Privacy contract implemented (consent, no Aadhaar, access log); legal review before any real pilot — `AI` + `YOU`

## Phase H — Paper 1 assembly (runs alongside everything)

- [ ] H.1 **Redo the editorial revision** on the paper, since the other agent's output is not on this machine: passive voice, students first and Harkiran Kaur last, `[1][4]` citations, v2 text carried into v3 — `AI`
- [ ] H.2 **Elaborate illustrated figures** in the style of the supplied sample (architecture, methodology, hardware); show drafts to the supervisor — `AI`, then `YOU` to show them
- [ ] H.3 Section drafts written from results as each phase finishes (2 Related Work already drafted) — `AI`
- [ ] H.4 Authors rewrite each section in their own words; the AI use is declared per Springer policy — `YOU`
- [ ] H.5 Verify all references again (24–25 in v2 plus the new ones) — `AI`
- [ ] H.6 Re-run the college AI/similarity checker and record the score — `YOU`
- [ ] H.7 Confirm the **venue, template and page limit** with the guide; author emails (Harkiran Kaur's and two students') — `YOU`
- [ ] H.8 Declarations: funding, competing interests, ethics, data and code availability, author contributions — `BOTH`
- [ ] H.9 Final number check (every number from `numbers.py`), compile, supervisor review, submit — `BOTH`

## Phase I — Integration and release

- [ ] I.1 End-to-end run on one command (Docker Compose): data → forecast → storage → advice → plan → approval → WhatsApp → sale → stock — `AI`
- [ ] I.2 Whole-system backtest vs simple practice, per season and scenario — `AI`
- [ ] I.3 Demo script run from a fresh clone on a second machine; demo video — `BOTH`
- [ ] I.4 Tag releases, archive code and data scripts (e.g. Zenodo DOI), user manual, deployment guide — `AI`

## Phase J — Real-world validation (Paper 2 and any pilot)

- [ ] J.1 Talk to a Patiala FPO CEO and an arhtiya (the question list is in the FPO profile) — `YOU`
- [ ] J.2 Ethics approval from the college before any human testing — `YOU`
- [ ] J.3 Usability and message-comprehension tests (small, with consent) — `BOTH`
- [ ] J.4 Hosting choice, DPDP review, then an FPO pilot — `YOU`

## Suggested order for the next sessions

1. **Now:** Phase 0 (one short session) and Phase A.1 downloads (you type CAPTCHAs; about 17 rounds).
2. **In parallel, in separate sessions:** E (finish allocation experiments), H.1–H.2 (editorial redo and elaborate figures), A.3 (MSP table), G.1 (remaining contract freezes).
3. **After A and B:** C, then D.
4. **After E and D:** G.4–G.5, then G.6 once you have the WhatsApp test number.
5. **Last:** I, then H.4–H.9.

## What to say to continue

Pick the next unticked step and say its number, for example "do 0.4 and 0.5" or "start E.1". Everything above is already linked to its detailed card or guide.
