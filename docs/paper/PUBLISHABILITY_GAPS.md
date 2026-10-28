# What is still needed to make the paper definitely publishable, phase by phase

**Update 28 Oct 2026:** the C.0 protocol was approved and frozen, the comparison was run once (no method beat the naive forecast), the prospective check, sensitivity and scaling grids, the IMD citation (checked against the PDF) and a blank-date plausibility check are done, and the paper is rewritten to 12 pages in `kisaniti_research_paper/paper/drafts/kisaniti_v4_final_2026-10-28.pdf`. Still open and not doable by the AI: the hand check against the portal (CAPTCHA), a native-speaker review, the authors' own pass and sign-off of the AI-use declaration, the similarity check, and confirming the venue's page limit (12 pages incl. references was assumed). The tables below are the 27 Oct state.

27 October 2026 • Companion to [PUBLICATION_PLAN_V3.md](PUBLICATION_PLAN_V3.md). "Me" is the AI agent working in the repositories; "you" is the owner or the authors. Status words: DONE (script, result file, test), RUNNING, TODO-ME, NEEDS-YOU.

## 1. Data in (farmer records, mandi prices, routine data, hardware)

| Need | Why it matters for acceptance | Who | Status |
|---|---|---|---|
| Hand check of the 56-row sheet; independent spot check against a second source | A single-source dataset must show that its parsing and inferred dates are right | you (CAPTCHA site) | NEEDS-YOU; sheet ready |
| Decide the blank-date rule and the sub-yard merge | They change about 9% (blank dates: 6,098 vs 6,678 rows) and about 20% (sub-yards) of index days | you | NEEDS-YOU; both run as labelled sensitivity variants |
| More seasons: Oct 2026 onward | The prospective check is the only truly unseen test | you (monthly CAPTCHA download) | NEEDS-YOU; protocol treats it as a separate partial check |
| Transport costs and mandi distances from real sources | The allocation uses invented rates; sensitivity shows what that does | you / FPO | NEEDS-YOU; sensitivity grid RUNNING (me) |
| Real sensor week in grain | Turns "feasibility on simulation" into feasibility on data | you (kit) | NEEDS-YOU |
| Product import job for the raw files | Cleaning is reproducible only in the research repo today | me | TODO-ME, small |

## 2. Cleaning, verification, training and comparison

| Need | Who | Status |
|---|---|---|
| Approve and freeze the C.0 protocol (with the corrected pair counts and the eight open choices) | you | NEEDS-YOU |
| Install the pinned model packages (statsforecast, LightGBM, Chronos-2) and lock them; wire models 4–8 into the evaluation kit; write the grids file | me | after approval (the packages are not in the research environment today) |
| A runner that refuses to run unless the freeze commit exists, the code and data are unchanged and the test season has not been run; results with commit hash and data checksum | me | DONE: `run_protocol.py` and `protocol_gate.py`, 16 tests, refuses today because nothing is frozen |
| Validation run, configuration freeze, single test-season run, prospective check, ranges and coverage, tables and figures | me | after approval |
| Cleaning comparison (MAD, Hampel, IQR) | me | descriptive part DONE (screens flag 2.2% to 8.8% of rows and would move 27 to 110 of 433 index days of 1121); precision and recall need the hand check |
| Reference check: every DOI against Crossref/DataCite | me | DONE: 38 match, 5 author-name formatting flags, 18 entries without DOI to check by hand |

## 3. Optimizer

| Need | Status |
|---|---|
| Solver vs brute-force oracle on random tiny cases | DONE: 1,728 of 1,728 cases agree (3 solvers, 2 methods, 324 instances, one proven infeasible) |
| Policy comparison on 60 plans | DONE (seven policies, all plans validated, exact solves proven optimal) |
| Sensitivity to transport rate and mandi-day limit (3 x 3 grid) | RUNNING |
| Scaling grid, 10 to 1,000 farmers, three solvers and greedy | script written, runs after the grid (timings need an idle machine) |
| Weighted-sum policies at the 300 s limit | NEEDS-YOU (decision E.0), then about 1 to 2 hours of compute |
| Re-run with basmati prices on the training seasons only (2021-22 to 2023-24), so the test seasons stay unseen | TODO-ME |

## 4. Approve, reject or edit

| Need | Who | Status |
|---|---|---|
| Independent validator accepts exactly the valid manager edits | me | DONE: 4,431 of 4,431 random edits get the same verdict as a second rule-based checker (1,052 valid, 3,379 invalid) |
| Parallel approvals never over-reserve stock | me | DONE in tests (8 parallel approvals, two competing plans); to be cited from the evidence file |
| Real approver roles and reason codes | you / an FPO | NEEDS-YOU; the paper states them as demo policy |

## 5. Messages out, replies and sales back

| Need | Who | Status |
|---|---|---|
| Exact rendering of every text in three languages | me | DONE in tests (hand-written expected strings) |
| Native-speaker review of the Punjabi and Hindi texts | you | DONE 28 Oct: English, Punjabi and Hindi reviewed by independent reviewers (names to be recorded for the acknowledgments) |
| One real message to the authors' own phone; screenshots for the paper | you | NEEDS-YOU (phone, Meta account for the official channel; the manager's own-WhatsApp link needs none) |
| Comprehension or usability study | you | NEEDS-YOU: ethics approval first; not claimed in the paper |
| Planned-against-actual loop on real sales | an FPO | not possible before a pilot; the paper says so |

## 6. The paper itself

| Need | Who | Status |
|---|---|---|
| Insert the single pre-registered result, whatever it is, and write what follows from it | me (draft), authors (words) | after the run |
| Cut 19 pages to the venue limit (about 11 to 12) | authors with me | after venue is confirmed |
| Cite the IMD normals; rewrite the basmati movement sentence to the computed values | me | TODO-ME |
| Authors' rewrite in their own words, declaration of AI use, similarity check, supervisor review | authors | NEEDS-YOU |

## 7. The one thing that decides everything

Without the forecast comparison on basmati the paper has a regime test, an allocation study and a system, but not its forecasting contribution. The comparison needs two things only you can give: the protocol approval, and (for the primary index variant) either the hand check or the choice of `dated_only` as primary.
