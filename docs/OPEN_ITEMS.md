# Open items — things that need something extra

24 October 2026 • One list of everything found during the B.1 / B.4 / C.0 work that is not yet settled. Each item has an owner (`AI` = an agent can do it, `YOU` = only the project owner can, `BOTH`), a status, and where the detail lives. Update the status here when an item closes; the phase checklist stays in [PROJECT_TRACKER.md](PROJECT_TRACKER.md). Paths starting `rp:` are in `~/Documents/kisaniti_research_paper`.

## A. Waiting for you

| ID | Item | Why it matters | Detail |
|---|---|---|---|
| O1 | Hand-check the 50 rows on AGMARKNET | Gate G1: confirms or rejects the blank-date rule (+9.5% rows); nothing is modelled before it | `rp:results/L2_cleaning/hand_check_instructions.md` |
| O2 | Approve the C.0 protocol and its deviations, then freeze it | Fixes models, splits, tests and claims before any test-season result exists | `rp:layers/L3a_forecasting/PREREGISTERED_PROTOCOL.md` §11 |
| O3 | Decide how product advice beyond 28 days is handled | Forecast evidence stops at 28 days (index exists Sep–Jan only), but CHAT-22 and hold-or-sell reach further | protocol §4 |
| O4 | Folders `kisaniti_final`, `untitled folder`, `untitled folder 2` and the older copy `sem_7/kisaniti` | Tracker 0.6; the `sem_7` copy confused the start of this session | tracker 0.6 |
| O5 | Supervisor: figure drafts, venue, page limit, author emails | Tracker H.2, H.7 | tracker |

## B. Data findings still to resolve

| ID | Finding | Next step | Owner | Status |
|---|---|---|---|---|
| O6 | Share of rows with min = max = modal rises from 7% (2024-25) to 27% (2025-26) and 56% (2026-27) | Ask in the hand check whether the website shows the same; decide whether min/max may be used at all; the index uses modal only | YOU (check) then AI | open |
| O7 | Very large arrivals at Khanna (about 18,000 t on many days; median over all rows 190 t) | One Khanna row (HC42) is in the hand-check sheet; arrivals are excluded from the primary models until checked | YOU | open |
| O8 | Price of ₹800 (Kot Ise Khan, 31 Oct 2021, 1 t) against a season median of ₹3,400 | HC02 in the sheet; flagged, not removed | YOU | open |
| O9 | Market map was **not** built in B.1. 86 raw names include 9 groups that look like sub-yards of one town (for example six Bhikhiwind names, three Samana names), and the index's "at least 3 mandis" rule currently counts each raw name | Decide with a rule (merge sub-yards or not). **Sensitivity measured (24 Oct, `build_index.py` `*_towns` variants):** index days barely change (1121: 433 to 431; 1509: 369 unchanged), but about 20% of days get a different value (median difference ₹0, largest ₹617 = 29%, on days with few mandis). So the rule matters and must be chosen and reported | AI built the variants, rule needs YOU | open |
| O10 | Two same-variety, same-day pairs (Bariwala 18 Nov 2025; Bareta 11 Sep 2026) | Hand-check rows HC40/41, HC44/45; excluded from the index meanwhile | YOU | open |
| O11 | Outlier screens (MAD, Hampel, IQR) not implemented; no literature says which suits AGMARKNET | Implement as candidates, score on the hand-checked sample and by forecast effect (B.3) | AI after O1 | waiting |
| O12 | Jain 2020 fills missing days with a spline; our rule forbids showing filled values as observed | Compare in B.3, do not adopt silently | AI | waiting |
| O13 | CEDA long history (A.4) stays on hold by your decision | Re-test the portal later; the paper states the limit | — | on hold |

## C. Literature

| ID | Item | Status |
|---|---|---|
| O14 | Ladhar et al. 2023 (soybean cooperatives, forecast + sell-or-wait, field-tested app) may contradict the review's gap 4 on hold-or-sell | worker running: `rp:literature/LADHAR2023_CHECK.md` |
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
| O23 | A.3 official MSP table with source links | `rp:layers/L1_data/msp/` | running |
| O14 | Ladhar 2023 overlap | `rp:literature/LADHAR2023_CHECK.md` | running |
| O24 | Can Chronos-2 run here at all? (protocol model 8) | report only | running |
| O25 | B.2b second source for the independent spot check | `rp:layers/L2_cleaning/SPOT_CHECK_SOURCES.md` | running |

## F. Not started

- E allocation experiments (scripts exist, never run).
- D hold-or-sell and G product layers.
- Hardware kit (F.3) and the WhatsApp test number (G.6).
- Later months: new Oct–Dec 2026 downloads would extend the unseen 2026-27 season; the index and protocol would then be rerun, not changed.
