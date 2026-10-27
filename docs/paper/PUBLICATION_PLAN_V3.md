# Publication plan for the one paper: what is done, what is verified, what must happen, in order

27 October 2026 • Written at the owner's request ("do all the work first so I can make a good publishable paper; everything verified"). It sits under [THE_ONE_PAPER.md](THE_ONE_PAPER.md) (thesis, contribution wording, page budget; CHAT-33/34) and does not change it. Every status below was checked against the files on 27 Oct; the checkable numbers are re-checked by `kisaniti_research_paper/scripts/verify_claims.py` (output `paper/claims_status.csv`).

## 1. The honest position

| # | Contribution (THE_ONE_PAPER §2) | Evidence today | Strength it may claim | Blocked by |
|---|---|---|---|---|
| 1 | Regime test: MSP share separates price-bound from market crops | Paddy (Common) 2020–25: lowest seasonal MSP share 96.1%, MSP rule has the lowest forecast error (Rs 1.26 per q); reproduced from the v2 result files (222 of 222 numbers). **Basmati contrast:** index exists (433 and 369 days) but the sentence "median change Rs 50, gap Rs 1,468" is **not reproduced** (§4) | "shows" for paddy. The basmati half needs a corrected sentence | rewording only |
| 2 | Fair cooperative allocation, price of fairness, solvers | Exact solver checked against brute force in 563 core tests; 60-plan policy comparison run on the synthetic persona at real 2025 prices, all plans validated, exact solves proven optimal (kisaniti: min fulfilment 68.6%, Gini 0.0019, price of fairness Rs 5.7/q; greedy has the same fairness but gives up Rs 14.1/q) | "shows" for method properties; synthetic, never "improves incomes" | weighted-sum policies hit the 300 s limit (decision E.0); scaling grid, sensitivity and the basmati re-run (E.4) not run |
| 3 | Forecasting with ranges for basmati | Index, protocol draft, evaluation kit tested on synthetic series. **No model has been run on the prices.** | nothing yet | your hand check (B.2), protocol approval (C.0), then one pre-registered run |
| 4 | Hold-or-sell with ranges | nothing | exploratory only | needs 3; parameter sources (storage loss, holding cost) |
| 5 | Low-cost storage sensing (ESP32 + SHT30) | Rule, firmware, simulated scenarios, software node and pipeline | "feasibility" | the kit (F.3) for any real reading |
| 6 | Working system | Whole flow on fictional farmers, 14-step chain, 319 backend tests, manager's WhatsApp hand-over, CI green | "demonstrated" | screenshots, a real test message to the authors' own phone |

## 2. Gates, in order (a later gate never starts before the earlier one is passed)

| Gate | What | Who | Evidence that it passed |
|---|---|---|---|
| G1 | Hand check of the 56-row sheet on the AGMARKNET site; decides the blank-date rule | **you** (the site needs CAPTCHAs; I cannot and will not bypass them) | filled sheet committed |
| G2 | Approve the C.0 protocol with the corrections in §3, then a commit whose message starts "C.0 freeze" | **you** approve, I commit | the commit |
| G3 | Install the pinned model packages, write the grids file, run validation (2024-25) only | me | grids file committed before the first run |
| G4 | Freeze the chosen configuration (a commit) | me | the commit |
| G5 | Run the test season 2025-26 **once**, then the prospective 2026-27 check separately | me | results files with commit hash and data checksums |
| G6 | Paper numbers only through `paper/numbers.py`; `verify_claims.py` shows no FAILED | me | the script output |
| G7 | Writing: authors rewrite in their own words, declare AI use as the venue requires, reference DOIs opened, similarity check, supervisor review | **authors** | checker report; declarations |

## 3. What must be corrected before G2 (found on 26–27 Oct)

1. **Protocol §4 pair counts ignore §3's own eligibility rule.** After the rule, Basmati 1509 has 57 and 51 test pairs at 7 and 14 days (not at least 60) and 38 at 28 days (under the 40-pair floor). Options in `layers/L3a_forecasting/EVALKIT_ASSUMPTIONS.md`.
2. **Eight points where the draft is silent** (loss for the DM test, whether the origin day counts, which calendar day seasonal naive uses, and so on): `EVALKIT_ASSUMPTIONS.md` lists my choices; confirm or change them.
3. **Primary index variant depends on G1.** If you want the model comparison to start before the hand check, the only choice that needs no inferred dates is `dated_only` as primary with `with_blank` as a labelled sensitivity run. That is an amendment to §2 and is yours to decide.
4. **E.0, the solver time limit.** The three weighted-sum policies ran the full 300 s in all 6 pilot runs (not proven optimal). Recommendation: keep 300 s, report the termination status of every run, and say in the paper that those three policies are time-limited. A full run is then about 60 × 3 × 300 s of CPU (about 2.5 hours on 6 workers).

## 4. Verification findings (re-checked today)

| Claim | Result |
|---|---|
| 10 numbers from the v2 abstract and the protocol (MSP shares, Rs 1.26, Rs 0.79, Rs 3.5–13.9, every-run violations, index days, 6,678 rows = 6,098 + 580, 56-row sheet, detectable gains) | **verified** against result files |
| "Basmati median daily change Rs 50; same-day gap about Rs 1,468 for 1121" (THE_ONE_PAPER §1) | **fails**: the index gives a median daily change of Rs 100 for 1121 (Rs 55 for 1509) and a same-day gap across mandis of median Rs 950, mean Rs 997, maximum Rs 2,750. The sentence must be reworded to the computed values and their definitions before it enters the paper |
| Protocol §4 "at least 60 test pairs per primary horizon" | **fails** for Basmati 1509 (§3.1) |
| "A forecasting model beats naive on basmati" | **not run** |

The 27 Oct file `paper/revisions/2026-10-27/` (v3 "corrected", docx and pdf) is the paddy paper revised again; its text has not been audited here and its numbers are not yet in the claims ledger. Next step is to add every number in it to `verify_claims.py`.

## 5. Work packages after the gates (my part)

| WP | Work | Starts after | Done when |
|---|---|---|---|
| W1 | Wire models 4–8 (ETS, ARIMA, LightGBM, GRU, Chronos-2) into the evaluation kit; pin the packages (the research environment has none of them today: the protocol's Chronos run was in another environment); grids file | G2 | synthetic tests green; versions in the lock |
| W2 | `run_protocol.py` that refuses to run unless the freeze commit exists and the tree is clean, and writes results with commit and checksums | G2 | a dry run on synthetic data passes and a run without the freeze fails |
| W3 | Validation run, configuration freeze, single test run, prospective check, intervals and coverage, tables and figures | G3–G5 | results files exist; `numbers.py` reads them |
| W4 | Hold-or-sell backtest, exploratory (needs W3 and sourced parameters) | W3 | reported per season, no significance claim |
| W5 | Allocation: scaling grid, sensitivity, basmati prices (E.4), weighted-sum policies | E.0 | figures from scripts |
| W6 | Claims ledger for every number in every section; reference verification log (57 entries on file, all opened once; re-open each cited DOI before submission) | continuous | no FAILED, no CANNOT_CHECK |
| W7 | Section drafts from the results, in the WRITING_GUIDE style, marked as drafts for the authors' rewrite | W3, W5 | authors' own pass (G7) |
| W8 | Real hardware week, screenshots, one real WhatsApp message to the authors' phone | kit, Meta account | feasibility claim written from real data |

## 6. What I will not do

- Run any model on the price index before G2 (that would break the pre-registration the paper relies on).
- Bypass CAPTCHAs, invent numbers, data or references, or state results that no script produced.
- Write the final paper text as if it were the authors' own words; drafts are labelled, and AI assistance is declared as the venue's policy requires.

## 7. What I need from you, in the order it unblocks work

1. G1: the 56-row hand check (the sheet and `hand_check_instructions.md` are ready).
2. G2: approve the protocol with the corrections in §3 (one sentence is enough), including the primary-variant rule.
3. E.0: keep 300 s?
4. B.1d: merge sub-yards of one town or not (affects about 20% of index days; the protocol already runs it as a sensitivity variant).
5. Kit purchase and the Meta account (hardware week and the real message).
6. Venue, template and page limit confirmed with the supervisor; ethics approval before any human testing.
