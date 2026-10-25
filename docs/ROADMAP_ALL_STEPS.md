# Roadmap: every step of every phase, done and left

26 October 2026 • Generated from one list, so the counts match the rows. Phases 0 to J are the tracker's; K to P cover the login, hardware software, website, deployment, WhatsApp and quality work added since. `H-n` with a hyphen is hardware integration software; `H.n` with a dot is the paper. Status: **done**, **started**, **not started**, **on hold**, **waiting** (needs another step). Owner: `AI`, `YOU`, `BOTH`.

**Total: 125 steps. Done 24, started 10, not started 85, on hold 2, waiting 4.**

| Phase | Steps | Done | Started | Not started | On hold / waiting |
|---|---|---|---|---|---|
| 0. Housekeeping | 6 | 5 | 0 | 1 | 0 |
| A. Data (Layer 1) | 6 | 3 | 0 | 1 | 2 |
| B. Cleaning (Layer 2) | 7 | 1 | 2 | 4 | 0 |
| C. Forecasting (Layer 3a) | 8 | 0 | 1 | 7 | 0 |
| D. Hold-or-sell (Layer 3c) | 4 | 0 | 0 | 4 | 0 |
| E. Allocation and fairness (Layer 3d) | 6 | 0 | 1 | 5 | 0 |
| F. Hardware (your part) | 6 | 1 | 0 | 5 | 0 |
| G. Product layers | 8 | 0 | 2 | 6 | 0 |
| H. Paper 1 | 10 | 1 | 2 | 7 | 0 |
| I. Integration and release | 4 | 0 | 1 | 3 | 0 |
| J. Real-world validation | 4 | 0 | 0 | 4 | 0 |
| K. Login, FPO separation and languages | 11 | 5 | 0 | 6 | 0 |
| L. Hardware integration software (H-n steps) | 12 | 5 | 0 | 6 | 1 |
| M. Website pages (demo path) | 10 | 1 | 0 | 6 | 3 |
| N. Deployment and operations | 8 | 1 | 0 | 7 | 0 |
| O. WhatsApp (Layer 5) | 7 | 0 | 0 | 7 | 0 |
| P. Quality and traceability | 8 | 1 | 1 | 6 | 0 |

## Phase 0: Housekeeping

Clean up and reconcile.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| 0.1 | Review everything done so far | done | AI |  |
| 0.2 | File the 12 basmati CSVs with manifests | done | AI |  |
| 0.3 | Apply editorial decisions to the writing guide | done | AI |  |
| 0.4 | Commit hand-off notes and the editorial decision | done | AI |  |
| 0.5 | Fix stale status rows | done | AI |  |
| 0.6 | Decide what to do with the extra folders in ~/Documents (kisaniti_final, untitled folders, old sem_7 copy) | not started | YOU | OPEN_ITEMS O4 |

## Phase A: Data (Layer 1)

Get and file the price data.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| A.1 | Download all 29 basmati months | done | BOTH | 6,098 rows, 86 mandis |
| A.2 | Coverage report | done | AI |  |
| A.3 | Official MSP table with source links | done | AI | 18 values; I re-opened all 12 PIB pages. The tracker box was not ticked until now |
| A.4 | Long history (CEDA) for forecasting power | on hold (your decision) | YOU | your decision: no email or registration; retry only if the site works |
| A.5 | data.gov.in registration for the live demo | on hold (your decision) | YOU | your decision |
| A.6 | Extras: Haryana mandis, weather, export prices | not started | AI | optional, future work |

## Phase B: Cleaning (Layer 2)

Turn raw files into a checked dataset.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| B.1 | Tidy parser, mechanical checks, row tracing, tests | done | AI | 6,678 rows reconcile with all manifests; commit ee9de51 |
| B.1d | Market map: decide whether sub-yards of one town are merged (affects about 20% of index days) | not started | BOTH | OPEN_ITEMS O9; variants already built |
| B.2 | Hand check of the 56-row sheet on the AGMARKNET site (decides the blank-date rule) | not started | YOU | sheet and instructions ready |
| B.2b | Independent spot check against a second source | started | BOTH | sources researched; eNAM must be opened by you in a browser |
| B.3 | Compare cleaning options (MAD, Hampel, IQR, forward fill) on the hand-checked sample and by forecast effect | not started | AI | waits for B.2; literature checked |
| B.4 | Variety-level daily index | started | AI | built both ways; primary one chosen by B.2 |
| B.5 | Dataset release with exact reconciliation | not started | AI |  |

## Phase C: Forecasting (Layer 3a)

Compare models on a pre-registered protocol.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| C.0 | Pre-registered protocol | started | BOTH | drafted; needs your approval, then a freeze commit |
| C.1 | Freeze the forecast contract slice (ranges beyond 7 days) | not started | AI |  |
| C.2 | Baselines: naive, seasonal naive, moving average | not started | AI | (no MSP rule for basmati) |
| C.3 | ETS, ARIMA, LightGBM | not started | AI |  |
| C.4 | GRU and Chronos-2 zero-shot | not started | AI | Chronos-2 checked: it runs on this Mac |
| C.5 | Prediction ranges and coverage check | not started | AI |  |
| C.6 | Rolling-origin evaluation on the held-out season, tests, figures | not started | AI |  |
| C.7 | Promotion rule: best model per variety and horizon, or 'baseline is enough' | not started | AI |  |

## Phase D: Hold-or-sell (Layer 3c)

Advice on when to sell.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| D.1 | Rule in the shared core with tests | not started | AI | needs its contract slice first |
| D.2 | Parameter table: storage loss, holding cost, interest | not started | AI |  |
| D.3 | Backtest against sell-at-harvest, fixed month, median-forecast, perfect foresight | not started | AI | needs C |
| D.4 | Sensitivity and scenario seasons | not started | AI |  |

## Phase E: Allocation and fairness (Layer 3d)

Fair plans across members.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| E.0 | Decide the solver time limit for the weighted-sum policies | not started | BOTH | OPEN_ITEMS O29 |
| E.1 | Policy comparison on the 300-member persona | started | AI | one-seed smoke run finished (31 min); results not final; finding: paddy prices are constant |
| E.2 | Solver comparison and scaling grid | not started | AI |  |
| E.3 | Price of fairness and sensitivity | not started | AI |  |
| E.4 | Re-run with basmati prices (where prices differ across mandis) | not started | AI | needs B |
| E.5 | Figures and paper sections 3.6 and 4.4 | not started | AI |  |

## Phase F: Hardware (your part)

Build and test the real node, following HARDWARE_BUILD_SPEC.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| F.1 | Storage rule, firmware, Wokwi twin, simulations | done | AI |  |
| F.2 | Observe the Wokwi MQTT leg end to end | not started | BOTH |  |
| F.3 | Buy the kit | not started | YOU | longest wait; order first |
| F.4 | Wire, flash, calibrate with salt, settling and outage tests | not started | YOU | needs F.3 |
| F.5 | One-week run in grain | not started | YOU | needs F.4 |
| F.6 | Optional moisture-meter check (design conflict to settle) | not started | YOU | HARDWARE_BUILD_SPEC section 2 |

## Phase G: Product layers

Contract first, then code, tests, documents.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| G.1 | Freeze contract slices. Frozen: shared, sensor L3b, allocation L3d, access, devices. Left: L1, L2, L3a, L3c, L4, L5 | started | AI |  |
| G.2 | L1/L2: import job, data and cleaning screens | not started | AI | needs G.1 and B |
| G.3 | L3a/L3c: price and advice screens with ranges | not started | AI | needs C and D |
| G.4 | L3d: plan generation and proposal screens | not started | AI | needs E |
| G.5 | L4: review, edit, approve with exact stock reservation, release; concurrency tests | not started | AI | login (K) is built |
| G.6 | L5: WhatsApp templates, J-form sale entry (see phase O) | not started | BOTH |  |
| G.7 | UI polish and restyle | not started | YOU | you said you will change the UI later |
| G.8 | Privacy contract implemented: consent, no Aadhaar, access log | started | BOTH | sign-in audit log done; consent and phone masking not; legal review before any pilot |

## Phase H: Paper 1

One Springer paper, every number from a script.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| H.1 | Editorial revision of v2 | done | AI |  |
| H.2 | Illustrated figures | started | AI | drafts done; supervisor to review |
| H.3 | Section drafts from results | started | AI | related work drafted; results sections wait for C, D, E |
| H.4 | Authors rewrite in their own words; declare AI use | not started | YOU |  |
| H.5 | Verify all references; add the new ones after reading their full text | not started | AI | 8 candidates found, 2 read in full |
| H.6 | Re-run the college similarity checker | not started | YOU |  |
| H.7 | Confirm venue, template, page limit, author emails | not started | YOU |  |
| H.8 | Declarations | not started | BOTH |  |
| H.9 | Final number check, compile, supervisor review, submit | not started | BOTH |  |
| H.10 | Reword the novelty gap on hold-or-sell and cite Ladhar 2023 | not started | BOTH | OPEN_ITEMS O14 |

## Phase I: Integration and release

The whole chain on one command.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| I.1 | End-to-end run: data, forecast, storage, advice, plan, approval, message, sale, stock | started | AI | smoke scripts exist for login, devices and storage; the full chain does not |
| I.2 | Whole-system backtest against simple practice | not started | AI |  |
| I.3 | Run the demo from a fresh clone on a second machine; demo video | not started | BOTH |  |
| I.4 | Tag releases, archive with a DOI, user manual, deployment guide | not started | AI |  |

## Phase J: Real-world validation

Future work, not needed for the paper.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| J.1 | Talk to a Patiala FPO CEO and an arhtiya | not started | YOU |  |
| J.2 | Ethics approval before any human testing | not started | YOU |  |
| J.3 | Usability and message-comprehension tests | not started | BOTH |  |
| J.4 | Hosting choice, DPDP review, pilot | not started | YOU |  |

## Phase K: Login, FPO separation and languages

Built this session.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| K.1 | Decisions: login required, Punjabi and English, sign-in design (CHAT-35 to 39) | done | BOTH |  |
| K.2 | Access contract frozen (access/v1) | done | AI |  |
| K.3 | Backend: sign-in, sessions, CSRF, throttling, roles, two walls between FPOs | done | AI | 82 tests then; database wall proven independently |
| K.4 | Website: sign-in, change password, shell, users page, status | done | AI |  |
| K.5 | Punjabi and English text catalogs with a CI check | done | AI | Punjabi is my draft |
| K.6 | Native-speaker review of the Punjabi text | not started | YOU | not to be shown to real staff before this |
| K.7 | Restyle the interface | not started | YOU | later, by you |
| K.8 | Decide on automated browser tests (Playwright, a new dependency) | not started | YOU |  |
| K.9 | Second factor and breached-password check | not started | AI | future, needs your go-ahead |
| K.10 | A devices.manage capability and device audit events (access v2) | not started | BOTH | OPEN_ITEMS O37, O38 |
| K.11 | Hindi interface | not started | BOTH | optional |

## Phase L: Hardware integration software (H-n steps)

How the node plugs in without rework.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| H-1 | Conformance harness | done | AI |  |
| H-2 | Software node (simulator) | done | AI |  |
| H-3 | Readings in PostgreSQL | done | AI |  |
| H-4a | Device registry with tenant isolation; registry for kn-ingest from the database | done | AI | 106 tests then |
| H-4b | Broker credentials issued from the website; provisioning script that writes secrets.h | not started | AI | HW-P1 |
| H-5 | Scheduled storage assessment with persisted hysteresis | done | AI | 122 tests |
| H-6 | Devices and storage page in both languages | not started | AI | NEXT |
| H-7 | Firmware hardening: flash queue, time sync, update method | not started | BOTH | over-the-air decision is yours |
| H-8 | Real-node acceptance on your build | waiting for another step | YOU | needs F.4 |
| H-9 | Guide for adding a second sensor type | not started | AI |  |
| HW-P2 | Acceptance report generator from serial log, conformance output and calibration log | not started | AI |  |
| H-10 | Alert when a storage category changes; recompute option for late readings | not started | AI | OPEN_ITEMS O40, O41 |

## Phase M: Website pages (demo path)

What the demo shows.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| D0 | System status | done | AI |  |
| D1 | Market data and reconciliation | not started | AI | needs L1/L2 slices |
| D2 | Prices and forecast with ranges | waiting for another step | AI | needs C |
| D3 | Storage (the H-6 page) | not started | AI | NEXT |
| D4 | Hold or sell | waiting for another step | AI | needs D |
| D5 | Plan | waiting for another step | AI | needs E and G.4 |
| D6 | Manager approval | not started | AI | needs G.5 |
| D7 | Messages | not started | AI | needs phase O |
| D8 | Sale and stock | not started | AI | needs G.5, G.6 |
| D9 | Evidence page: tables, protocol, test and CI status | not started | AI |  |

## Phase N: Deployment and operations

Where it runs and how you reach it.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| N.1 | Local stack with one command, shown in the app | done | AI |  |
| N.2 | Choose the host and the account owner; student verification; read the credit validity | not started | YOU | HOSTING plan |
| N.3 | Production settings, HTTPS (Caddy), broker, deploy script | not started | AI |  |
| N.4 | URL: domain from the student pack or provider name | not started | BOTH |  |
| N.5 | Run the whole-stack scripts against the public URL | not started | AI | needs N.2, N.3 |
| N.6 | Backups and monitoring | not started | AI |  |
| N.7 | Availability statement for the paper (URL, commit, date) | not started | BOTH |  |
| N.8 | Optional temporary public link from this Mac | not started | YOU | your decision |

## Phase O: WhatsApp (Layer 5)

Messages to farmers.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| O.1 | Meta developer account, test app, your number as recipient | not started | YOU |  |
| O.2 | Freeze the L5 contract slice | not started | AI |  |
| O.3 | Message channel with a preview-only implementation; templates in three languages as files | not started | AI |  |
| O.4 | Template approval; check Hindi and Punjabi support | not started | YOU |  |
| O.5 | Real channel and webhook | not started | AI | needs N.3 |
| O.6 | One real test message to your own number | not started | BOTH | only with your explicit yes |
| O.7 | Consent and opt-out records | not started | AI | with G.8 |

## Phase P: Quality and traceability

Everything tested, every bug traceable.

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| P.1 | CI with five jobs, including a real PostgreSQL and the whole stack | done | AI |  |
| P.2 | Bug register | started | AI | 6 entries so far; kept up to date |
| P.3 | Structured logs with request ID, user and FPO, no secrets | not started | AI |  |
| P.4 | Build identity (commit hash) in the API and the footer | not started | AI |  |
| P.5 | Screen-to-cause drill, recorded | not started | AI |  |
| P.6 | Automated browser tests (if you approve K.8) | not started | AI |  |
| P.7 | Demo-path script that fails on any broken step | not started | AI |  |
| P.8 | Test that no secret appears in any log line | not started | AI |  |

## What only you can do (open)

- **0.6** (0): Decide what to do with the extra folders in ~/Documents (kisaniti_final, untitled folders, old sem_7 copy) — OPEN_ITEMS O4
- **B.2** (B): Hand check of the 56-row sheet on the AGMARKNET site (decides the blank-date rule) — sheet and instructions ready
- **F.3** (F): Buy the kit — longest wait; order first
- **F.4** (F): Wire, flash, calibrate with salt, settling and outage tests — needs F.3
- **F.5** (F): One-week run in grain — needs F.4
- **F.6** (F): Optional moisture-meter check (design conflict to settle) — HARDWARE_BUILD_SPEC section 2
- **G.7** (G): UI polish and restyle — you said you will change the UI later
- **H.4** (H): Authors rewrite in their own words; declare AI use
- **H.6** (H): Re-run the college similarity checker
- **H.7** (H): Confirm venue, template, page limit, author emails
- **J.1** (J): Talk to a Patiala FPO CEO and an arhtiya
- **J.2** (J): Ethics approval before any human testing
- **J.4** (J): Hosting choice, DPDP review, pilot
- **K.6** (K): Native-speaker review of the Punjabi text — not to be shown to real staff before this
- **K.7** (K): Restyle the interface — later, by you
- **K.8** (K): Decide on automated browser tests (Playwright, a new dependency)
- **N.2** (N): Choose the host and the account owner; student verification; read the credit validity — HOSTING plan
- **N.8** (N): Optional temporary public link from this Mac — your decision
- **O.1** (O): Meta developer account, test app, your number as recipient
- **O.4** (O): Template approval; check Hindi and Punjabi support

## The next steps I will take, in order

1. H-6 devices and storage page (both languages)
2. P.3 and P.4 structured logs and build identity
3. P.5 recorded screen-to-cause drill
4. H-4b broker credentials and the provisioning script
5. G.1 freeze the L1, L2, L3a, L3c, L4 and L5 slices (each only after you approve its scope)
6. N.3 production settings and deploy script (so the URL step is only your account)
7. O.3 message channel with the preview implementation and the three-language templates
8. B.3 and C.1 to C.7 as soon as the hand check and the protocol approval are in
