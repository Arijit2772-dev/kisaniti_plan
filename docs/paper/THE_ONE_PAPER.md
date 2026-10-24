# The one paper: how everything fits (decision CHAT-33)

24 October 2026 • There is **one** paper and no Paper 2. Wherever older files say "Paper 1" they mean this paper; anything they placed in "Paper 2" is now either inside this paper, or a stated limitation and future work. This file says what the paper claims, how strong each claim is allowed to be, and how the work is organised to deliver it.

## 1. The thesis (one sentence)

> A selling framework for farmer organisations should first test whether a crop's price is set by policy (MSP) or by the market, then apply the right tool: coordination and fairness where prices are fixed, and range-aware forecasting plus storage-aware advice where they move.

This ties the pieces together instead of presenting five separate topics, and it matches what our own data shows: paddy (common) sits at MSP in more than 96% of reports (strong evidence, 16,201 rows over six seasons), while basmati prices move (median daily change ₹50/q; same-day gap across mandis of about ₹1,468/q for 1121).

## 2. Contributions, and how strong each may sound

| # | Contribution | Evidence available | Allowed wording |
|---|---|---|---|
| 1 | **Regime test:** share of reports equal to MSP identifies price-bound crops; MSP rule beats learned forecasters there | Paddy (Common) 2020–25, 16k rows (already reproduced: 222/222 numbers); wheat in the procurement window; basmati as the contrast | "shows" (strong, well powered) |
| 2 | **Fair cooperative allocation:** exact lexicographic optimisation with per-farmer leximin; verified against a brute-force reference; price of fairness; solver comparison | Real price spreads between mandis plus **synthetic** FPOs we can generate in any number, so the experiments are well powered; synthetic nature stated | "shows" for method properties; never "improves farmers' income" |
| 3 | **Forecasting with ranges for basmati:** a small pre-registered comparison (baselines, ETS/ARIMA, ridge or gradient boosting, Chronos-2, plus GRU and graph models as exploratory) | 5 seasons, 433 / 369 daily index points; **smallest detectable gain 11–17%** | Differences above that size are reported as findings; smaller ones as "not distinguishable"; a negative result ("the baseline is hard to beat") is a valid result |
| 4 | **Hold-or-sell advice with ranges** | 5 seasons = 5 outcomes | "exploratory, per season, no significance claim" |
| 5 | **Low-cost storage monitoring (ESP32 + SHT30)** and a risk rule from published mould limits | Simulated scenarios done; a real one-week grain run if the kit is bought | "feasibility"; states that it cannot detect spoilage; real data only if collected |
| 6 | **Working system:** manager approval with atomic reservation, WhatsApp messages in three languages, J-form sale record, one-command demo | Automated tests, screenshots, a test message to the authors' own phone | "demonstrated", never "evaluated with users" (no user study) |

Rule: every sentence in the paper uses the wording of its row. Anything beyond it goes to the limitations section.

## 3. What makes a single paper strong without new data

1. **Use every dataset we already have:** Paddy (Common) 2020–25 (regime evidence, 176 mandis), wheat Apr–May 2025 (procurement window), basmati 2021–26 (market regime).
2. **Spend the statistical budget carefully:** pre-register the forecasting protocol before touching the test seasons (at most 6–8 models, fixed splits, corrections for multiple comparisons, effect sizes with intervals); evaluate several horizons; use block bootstrap by week and season; keep **Sep–Oct 2026 as a genuinely unseen final test** (it was downloaded after the design was fixed).
3. **Put the well-powered work in front:** the regime finding and the allocation experiments carry the quantitative weight; forecasting and hold-or-sell are reported honestly at the strength the data allows.
4. **Validate the data:** a 50-row hand check and one independent spot check, since we use one source.
5. **Show the system really works:** CI, brute-force verification, the demo script run from a fresh clone, real hardware readings if possible.
6. **Limitations stated plainly** (modal price is not the farmer's price; synthetic FPO; five seasons; one sensor; WhatsApp assumed for all members; no user study).

## 4. The venue decides the scope: ask the supervisor first

Everything above does not fit in a typical 10–12 page conference paper. Decide now, because the writing plan differs:

| Venue type | Length | What goes in the main text | What goes elsewhere |
|---|---|---|---|
| Journal (for example *SN Computer Science* or *Agricultural Research*) | about 20–30 pages | Everything in section 2 | Reproducibility details in a supplement and the repo |
| Conference proceedings (LNNS/LNCS) | about 10–12 pages | Contributions 1, 2, 3 in full; 4 as a short subsection; 5 and 6 as one section each with one figure | A longer technical report in the repository (cited) |

I recommend asking whether a **journal** is acceptable, since the user wants a single, final paper. The text will be written in modules so either target works.

## 5. How the work is organised (all unchanged phases of the tracker, in this order)

1. **Cleaning** of the basmati data (hand check, blank-date rule, variety index), plus the MSP table. Finish allocation experiments in parallel.
2. **Pre-registered forecasting protocol**, then forecasting with ranges, then the hold-or-sell backtest.
3. **Hardware:** buy the kit now (delivery takes days) so the one-week real run happens before writing results; the Wokwi twin covers the gap.
4. **Product layers and WhatsApp** to the point where the full flow is demonstrated, with tests.
5. **Integration run** and the final paper pass: numbers from scripts, references re-checked, authors' rewrite, checker re-run, supervisor review.

## 6. Time estimate

Roughly four to six weeks of sessions if data work, allocation, hardware and writing run in parallel and the user's own tasks (hand check, kit, WhatsApp number, supervisor decision on venue) are done early. The longest wait is the hardware: order the kit first.
