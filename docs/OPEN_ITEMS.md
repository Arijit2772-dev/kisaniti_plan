# Open items — things that need something extra

24 October 2026 • One list of everything found during the B.1 / B.4 / C.0 work that is not yet settled. Each item has an owner (`AI` = an agent can do it, `YOU` = only the project owner can, `BOTH`), a status, and where the detail lives. Update the status here when an item closes; the phase checklist stays in [PROJECT_TRACKER.md](PROJECT_TRACKER.md). Paths starting `rp:` are in `~/Documents/kisaniti_research_paper`.

## A. Waiting for you

| ID | Item | Why it matters | Detail |
|---|---|---|---|
| O1 | Hand-check the 50 rows on AGMARKNET | Gate G1: confirms or rejects the blank-date rule (+9.5% rows); nothing is modelled before it | `rp:results/L2_cleaning/hand_check_instructions.md` **28 Oct: agents inspected all 56 rows on the live portal (56/56 modal prices agree; 20 differ in arrivals; 4 missing-as-zero; blank-date rule 21/22 confirmed beneath the preceding date, but NO merged cell). Your sign-off is pending: `rp:results/L2_cleaning/hand_check_summary.json`.** |  **28 Oct: owner signed off; dataset VALIDATED in the product database; see `rp:results/audit/AUDIT_2026-10-28.md`.**
| O2 | Approve the C.0 protocol and its deviations, then freeze it | Fixes models, splits, tests and claims before any test-season result exists | `rp:layers/L3a_forecasting/PREREGISTERED_PROTOCOL.md` §11 |
| O3 | Decide how product advice beyond 28 days is handled | Forecast evidence stops at 28 days (index exists Sep–Jan only), but CHAT-22 and hold-or-sell reach further | protocol §4 |
| O4 | Folders `kisaniti_final`, `untitled folder`, `untitled folder 2` and the older copy `sem_7/kisaniti` | Tracker 0.6; the `sem_7` copy confused the start of this session | tracker 0.6 |
| O5 | Supervisor: figure drafts, venue, page limit, author emails | Tracker H.2, H.7 | tracker |

## B. Data findings still to resolve

| ID | Finding | Next step | Owner | Status |
|---|---|---|---|---|
| O6 | Share of rows with min = max = modal rises from 7% (2024-25) to 27% (2025-26) and 56% (2026-27) | Ask in the hand check whether the website shows the same; decide whether min/max may be used at all; the index uses modal only | YOU (check) then AI | open |
| O7 | Very large arrivals at Khanna (about 18,000 t on many days; median over all rows 190 t) | Two largest-arrivals rows (Khanna 2026-01_punjab.csv:21, Budalada 2025-11_punjab.csv:97) are in the hand-check sheet; arrivals are excluded from the primary models until checked | YOU | open |
| O8 | Price of ₹800 (Kot Ise Khan, 31 Oct 2021, 1 t) against a season median of ₹3,400 | row 2021-10_punjab.csv:149 is in the hand-check sheet; flagged, not removed | YOU | open |
| O9 | Market map was **not** built in B.1. 86 raw names include 9 groups that look like sub-yards of one town (for example six Bhikhiwind names, three Samana names), and the index's "at least 3 mandis" rule currently counts each raw name | Decide with a rule (merge sub-yards or not). **Sensitivity measured (24 Oct, `build_index.py` `*_towns` variants):** index days barely change (1121: 433 to 431; 1509: 369 unchanged), but about 20% of days get a different value (median difference ₹0, largest ₹617 = 29%, on days with few mandis). So the rule matters and must be chosen and reported | AI built the variants, rule needs YOU | open |
| O10 | Two same-variety, same-day pairs (Bariwala 18 Nov 2025; Bareta 11 Sep 2026) | Both pairs are in the hand-check sheet (2025-11_punjab.csv:46/47, 2026-09_punjab.csv:76/77); excluded from the index meanwhile | YOU | open |
| O11 | Outlier screens (MAD, Hampel, IQR) not implemented; no literature says which suits AGMARKNET | Implement as candidates, score on the hand-checked sample and by forecast effect (B.3) | AI after O1 | waiting |
| O12 | Jain 2020 fills missing days with a spline; our rule forbids showing filled values as observed | Compare in B.3, do not adopt silently | AI | waiting |
| O26 | **48 rows have a modal price of ₹2,389 or ₹2,390, mostly Kot Ise Khan in Oct–Nov 2025, mostly variety 1121 (season median about ₹3,150).** ₹2,389 is the 2025-26 paddy Grade A MSP (verified, see O23). They look like the MSP entered as a price, not a market price. Found by the spot-check worker; I confirmed the count | 4 of them are in the hand-check sheet; if the website shows the same, add a rule that flags modal = MSP value, and test its effect on the index | YOU (check) then AI | open |
| O27 | The repo's demo constants use the KMS 2025-26 paddy MSP (2369/2389) although KMS 2026-27 (2441/2461) was announced on 13 May 2026 | Decide which season the demo means; no code changed | YOU | open |
| O13 | CEDA long history (A.4) stays on hold by your decision | Re-test the portal later; the paper states the limit | — | on hold |

## C. Literature

| ID | Item | Status |
|---|---|---|
| O14 | Ladhar et al. 2023 does contradict gap 4 as written (a collective-level, ML-based, field-tested sale-timing system). Worker read the authors' own copy (full text); partial overlap: it has forecasting and prospect-theory sale dates, but no storage loss or holding cost, no allocation across members, no fairness, no sensors, no approval step. **The gap sentence must be reworded and the paper cited; the review file is not yet edited** | DONE (check); edit pending (authors), `rp:literature/LADHAR2023_CHECK.md` |
| O15 | Six references found this session are not in `references.bib` (Leys 2013, Hampel 1974, Pearson 2016, Rousseeuw & Croux 1993, Chandola 2009, Jain 2020; plus Gonzalez 2025, Ladhar 2023) | AI, after full text is read for each; abstract-only ones stay uncited as evidence |
| O16 | Full texts not read: Gonzalez 2025 (MDPI blocked), Bhoi 2023, Ladhar 2023 (ACM blocked) | library access may be needed: YOU |
| O17 | The R1 review never recorded cleaning practice; the 57-reference list is otherwise unchanged | note only |

## D. Protocol and design decisions to record

Per the repository rules these go into the decision register only after you approve them.

| ID | Proposed decision | Status |
|---|---|---|
| O18 | No MSP rule for basmati (basmati has no MSP; anchoring is UNKNOWN) | proposed |
| O19 | T-GCN and A3T-GCN dropped from the Paper 1 comparison | proposed |
| O20 | Arrivals not used as a feature until checked | proposed |
| O21 | Horizons 90 and 120 days not evaluated; 60 descriptive only | proposed |
| O22 | Blank-date rule becomes primary only if at least 19 of 20 checked rows agree | proposed (threshold is mine; change if you want) |

## E. Checks being run in parallel (24 Oct)

| ID | Item | Output | Status |
|---|---|---|---|
| O23 | A.3 official MSP table with source links | `rp:layers/L1_data/msp/` | **done**: 18 values (6 paddy seasons x 2 grades, 6 wheat seasons) from PIB; I re-opened all 12 PIB pages myself and the numbers match (common-grade and wheat values read from the tables; Grade A values confirmed to appear on the page); the DES PDF cross-check used `curl -k` (certificate check off), not relied on |
| O14 | Ladhar 2023 overlap | `rp:literature/LADHAR2023_CHECK.md` | running |
| O24 | Can Chronos-2 run here at all? (protocol model 8) | report only | **done**: yes, CPU only, system Python 3.13; I re-ran the worker's script and got the same forecasts and quantile shapes. Irregular timestamps fail unless the series is reindexed to daily with NaN gaps. Training-data overlap with public price series is unconfirmed (model card defers to the paper, arXiv 2510.15821). Not yet tested on the real index, on MPS, or on more than one series. Weights (about 478 MB) are cached in `~/.cache/huggingface` |
| O25 | B.2b second source for the independent spot check | `rp:layers/L2_cleaning/SPOT_CHECK_SOURCES.md` | **done**: no fully independent, no-login daily source found for named Punjab mandis. Best options: eNAM viewed by you in a normal browser (blocked automated access; may overlap AGMARKNET), and Tribune/trade-press ranges as plausibility only. Punjab Mandi Board hosts do not resolve. 15 cells proposed for October 2025 (not yet reviewed by me cell by cell) |

## E2. Allocation smoke test (one seed, 26 Oct; not final, results files left uncommitted)

| ID | Finding | Consequence |
|---|---|---|
| O28 | Real 2025 Paddy(Common) modal prices at the five candidate mandis are about ₹2,389 (the MSP) on nearly every date; mean cross-mandi spread ₹0.80. "Perfect foresight" therefore equals the KisanNiti plan exactly in all 6 weeks | This experiment cannot show a price-driven benefit. Gains over simple rules come from transport cost, capacity and fairness only. Claims about choosing better mandis need the basmati prices (E.4), where prices do differ. State this in the paper rather than hiding it |
| O29 | The three weighted-sum policies hit the 300 s solver limit in all 6 weeks (status TIME_LIMIT / FEASIBLE_NOT_PROVEN) and give the same values as max-min; one seed took 31 minutes, so 10 seeds would take about 5 hours | Decide the time limit for the weighted sums and report them as not proven optimal; or lower the limit and say so |
| O30 | "Independent best choice" sends 100% of lots and violates mandi-day limits in every run, so its "price of fairness" is negative and meaningless | Report it as an infeasible baseline, not as a cheaper plan |
| O31 | Basmati rows with modal ₹2,389 (O26) coincide with this constant paddy price. Untested idea: some Oct–Nov 2025 "1121" rows may carry the MSP-level paddy price | Include in the hand check (4 rows are there); do not assume |

## H-steps progress (hardware integration plan, FINAL_DELIVERY_PLAN §4.3)

H-1 conformance harness, H-2 software node and H-3 PostgreSQL persistence are **done** (product repo commit `c91330d`; report `docs/reports/2026-10-26_H1-H3_simulator_conformance_postgres.md`). Change from the plan: no HTTP read endpoints and no per-lot queries yet, because tenant isolation needs the device registry (H-4). Next: H-4, H-5.

## G. Hosting, WhatsApp, hardware and the next build step (26 Oct)

Plan: [HOSTING_WHATSAPP_HARDWARE_PLAN.md](HOSTING_WHATSAPP_HARDWARE_PLAN.md).

| ID | Item | Owner | Status |
|---|---|---|---|
| O32 | **BACKEND DONE 26 Oct (CHAT-37, CHAT-38): contract frozen, login and isolation built and tested; sign-in page still to build; Punjabi text unreviewed.** Earlier: **PARTLY RESOLVED 26 Oct: login is required (CHAT-35), interface in Punjabi and English (CHAT-36).** Still open: sign-in method, roles, digits, default language; proposal in [AUTH_AND_LANGUAGE_PROPOSAL.md](AUTH_AND_LANGUAGE_PROPOSAL.md). Earlier text: how FPO managers sign in, and their roles. The shared contract requires every tenant-scoped endpoint to act for a "principal" (404 for another FPO's object, 422 for a mismatched `fpo_id`), but authentication and roles are not frozen (the spec says it does not freeze roles). No website-facing read endpoint, manager screen or approval can be built until this slice is decided. I will not invent one | YOU decide, then AI freezes the slice (G.1) | open |
| O33 | **DONE 26 Oct (CHAT-40, CHAT-41).** Proposed decision H-4a (no HTTP, no auth needed): tables `devices` (device_id, fpo_id, source_origin) and append-only `device_assignments` (the frozen DeviceAssignment fields plus fpo_id); `kn-ingest` reads the registry from PostgreSQL instead of `devices.json`; a reading's FPO is resolved from the assignment history at `measured_at` (contract IC-DOMAIN-004); an untrusted time is never guessed; tests prove two FPOs never see each other's readings at database and service level. Needs your approval, then a decision-register entry, then code | YOU approve | proposed |
| O34 | Account owner and hosting choice (Azure for Students VM recommended; see the plan §2) | YOU | open |
| O35 | Create the Meta developer account and test app; add your own number as a recipient; check whether Hindi and Punjabi templates are available | YOU | open |
| O36 | Facts not verified yet: how many recipients a Meta test number allows; Hindi and Punjabi template support; India per-message rates; whether Azure public IPs give a free DNS name; which root certificate the ESP32 needs for the broker's public certificate; DigitalOcean's student credit | AI while building, YOU in dashboards | open |

| O37 | Device registration and assignment are not in the audit trail (the frozen audit event list has no device events); needs an access slice version bump | AI after your go-ahead | open |
| O38 | Registering a device needs `users.manage`; a dedicated `devices.manage` capability is cleaner (access slice v2) | decision for the owner | open |
| O39 | Broker credentials for a new device are still made by hand (H-4b would issue them from the website) | AI | planned |
| O40 | Late-arriving readings do not change slots already assessed; an optional recompute is not built | AI, if wanted | open |
| O41 | Slot length (1 h) and catch-up cap (14 days) are constants, not settings; no alert on a category change | AI | planned |
| O42 | Storage-rule thresholds and the 48 h hold remain desk-research values until the real-node bench tests | YOU (hardware) | open |
| O43 | The imported price dataset stays `PENDING_HAND_CHECK` until your 56-row hand check (B.2) and the decision on rows without a date (O9); then I run `set-status --to VALIDATED` with your note. The prices page shows the warning meanwhile | YOU, then AI | open 28 Oct: needs only your written sign-off of the agents' inspection (O1). |  **28 Oct: owner signed off; dataset VALIDATED in the product database; see `rp:results/audit/AUDIT_2026-10-28.md`.**
| O44 | Phone numbers and consent exist since slice 4 (fictional numbers only). Still open: erasure and data-access screens, retention job, reviewed consent notice text, shared household phones | AI, YOU (notice text, legal) | open |
| O45 | No link from a sensor to a lot; no distance or transport-cost inputs for mandis (transport cost is UNKNOWN) | AI, YOU for the cost evidence | open |
| O46 | The prices page lists at most 200 rows without paging; staleness is shown only as the newest date and its age | AI | planned |
| O47 | Punjabi text of the slice-1 pages (members, lots, prices, sensors) needs the same native-speaker review as K.6 | YOU | open |
| O48 | DEMO policies in plan/v1 need your decision or evidence before any real use: increment (100 kg default), transport (entered by the manager, no model), no waiting cost, mandi basis (30 days, 5 days minimum), 2-hour age limit for a sensor assessment, 30 s solver time limit (E.0) | YOU | open |
| O49 | The rule for "no reliable signal" (CHAT-22) is undefined until EXP-04 is run; forecasts show NOT_ASSESSED. The 95% ranges on real prices are wide (see the slice 2 report) | AI after C.0 | open |
| O50 | The plan uses the naive forecast until the protocol (C.0) is frozen and run; seasonal naive waits for C.2 because the protocol's "same calendar day" is ambiguous | YOU (C.0), then AI | open |
| O51 | Whether a result that is not proven optimal may ever become a proposal (G09) is open; today it is stored as NOT_PROPOSED | YOU | open |
| O52 | Inactive members' lots are still planned; whether they should be excluded is undecided | YOU | open |
| O53 | "Request a change" from another person (for example staff asking the admin) needs a role workflow that is UNKNOWN; today staff can only save a revision for the admin to approve | YOU | open |
| O54 | Who may approve in a real FPO is UNKNOWN; the demo gives `plans.approve` to the FPO admin only. Reason-code lists for rejection and release are DEMO | YOU | open |
| O55 | Freshness thresholds (when a changed price, storage or stock should block approval) and same-day cutoff rules are UNKNOWN; today three non-blocking warnings need acknowledgement | YOU | open |
| O56 | Release is for the whole remaining reservation; releasing one assignment or lot, and partial approval, are not built | AI, if wanted | open |
| O57 | The validator's refusal messages after a bad edit are English technical sentences shown in both languages | AI | planned |
| O58 | No sale can consume a reservation yet (slice 5); until then a reservation can only be released | AI | planned |
| O59 | The Punjabi and Hindi message texts were written by me and are UNREVIEWED; a native speaker must read both templates before any real send (the real channel must refuse unreviewed variants) | YOU | open |
| O60 | Real WhatsApp send (S4.5) needs your Meta account, approved templates and the supported template-language codes; the status webhook (S4.6) needs a public HTTPS address | YOU, then AI | open |
| O61 | Whether planning must be blocked for a member without ADVICE consent (privacy contract PR-T02) is undecided; it would change plan/v1 behaviour | YOU | open |
| O62 | Mandi names in messages are the price source's English names, also in Punjabi and Hindi texts | AI, if wanted | planned |
| O63 | The off-plan sale reconciliation (the person names which reservations to give up) is a provisional DEMO policy; architecture §14 lists the real workflow as UNKNOWN until a real FPO shows what it does | YOU (with the FPO) | open |
| O64 | Transport costs actually paid are not recorded, so planned against actual is gross, not net | AI, if wanted | planned |
| O65 | A sale cannot be edited; a void does not restore a consumed reservation (a new plan is needed) | YOU | open |
| O66 | Farmers' replies are typed in by staff; the webhook source and any reading of replies wait for the real channel (O60) | AI | planned |
| O67 | S5.5 proper (forecast accuracy from real outcomes) needs the protocol C.0 frozen and real, not fictional, sales | YOU, then AI | open |
| O68 | The farmer comprehension and usability study (S5.6) needs ethics approval first | YOU | open |
| O69 | **Manager's own WhatsApp.** You want the FPO manager to send to all farmers "what he wants" from his own WhatsApp. This is a new idea, not in any decision. Options: (A) official Meta Cloud API with approved templates (the current plan; needs the Meta account, O60; the manager types only the filled-in fields, not free text); (B) "click to send" links (`wa.me/<number>?text=<approved text>`): the manager taps each farmer's link and presses Send in his own WhatsApp; no Meta account and no cost, works on any phone, but sending is by hand per farmer and the system gets no delivery status (it would record "manager says sent"); (C) both, B first. Free text typed by the manager is a further question: it breaks the exact-text checks, so it would be a separate, clearly labelled "manager note" on top of the approved instruction. Decide A, B or C and whether a manager note is wanted; I then freeze a contract amendment first. **Your preference (26 Oct, in chat): the manager reads the plan, presses OK (approve), then a separate "Send to farmers" button, then a yes/no confirmation, and the farmers get the Punjabi message through his own WhatsApp.** Limit to confirm with you: a personal WhatsApp cannot be made to message many people from one tap; with option B the confirmation opens a list of ready-made links and he taps Send once per farmer (the system records each as "manager says sent"). One tap for all needs option A | YOU | open (direction chosen, limit to confirm) |
| O70 | Evaluation kit built (research repo, `layers/L3a_forecasting/evalkit.py`); eight open choices in `EVALKIT_ASSUMPTIONS.md` need your confirmation before the C.0 freeze, and the draft protocol's pair counts (§4) ignore its own eligibility rule (Basmati 1509 has 57 and 51 test pairs at 7 and 14 days, not 60 or more). Decide how §4 is corrected | YOU | open |

## F. Not started

- E allocation experiments (scripts exist, never run).
- D hold-or-sell and G product layers.
- Hardware kit (F.3) and the WhatsApp test number (G.6).
- Later months: new Oct–Dec 2026 downloads would extend the unseen 2026-27 season; the index and protocol would then be rerun, not changed.
