# KisanNiti research and experiment plan

15 October 2026 • Status: **PLANNED — no experiment has been run.** This document says, for every layer, *what we compare, why those options, how we measure, which scenarios, and what result would change a decision*. It is the basis for the [research paper](../paper/KisanNiti_Paper_Outline.md).

Prerequisites: market-data access (issue [OI-05](../OPEN_ISSUES.md)), coding/research-script authorisation (OI-10), pinned versions (OI-11), Git (OI-09).

## 0. Rules for every experiment

1. **Real data only for accuracy claims.** Synthetic fixtures may test mechanics, never accuracy or benefit.
2. **No leakage.** Every forecast uses only data available before its forecast origin (rolling-origin evaluation). Scalers, imputers, feature selection and graph structure are fitted inside each training window.
3. **Same targets for all candidates.** Identical series, origins, horizons and units. Report coverage: series a model refused or could not forecast count against it.
4. **Untouched test period.** The last full season of each crop is held out until the final run; it is looked at once.
5. **Significance, not just averages.** Use paired tests on per-origin errors (Diebold–Mariano for forecasts; Wilcoxon signed-rank for paired policy outcomes), and bootstrap 95% confidence intervals.
6. **Reproducible.** Each run records code commit, data manifest checksum, package versions, seed, hardware and run time in `kisaniti_implementation/experiments/EXP-xx/<run-id>/report.md` (folder created when experiments start). Large data stays outside Git.
7. **Report negative results.** If a fancy method does not beat a baseline, the paper says so.
8. Record results as PASS/FAIL/NOT_RUN/BLOCKED against the decision criteria below. Never fill a paper table before the run exists.

## 1. Summary table

| ID | Layer | Question | Options compared | Main metrics | Decision it informs |
|---|---|---|---|---|---|
| EXP-00 | All | What is already known? | Literature: mandi price forecasting in India, MSP/procurement studies, grain storage monitoring, fair allocation, agri decision support | Structured review table | Paper related work; novelty claim |
| EXP-01 | L1 | What market data actually exists for our series? | data.gov.in daily, data.gov.in variety-wise, CEDA bulk history | Days covered, gaps, revisions, variety-name consistency, latency | Source choice; forecastable series list (OI-03, OI-05) |
| EXP-02 | L2 | Which cleaning rules help without deleting real spikes? | No screening; rules only; rules + MAD; rules + Hampel; rules + IQR | Precision/recall on a hand-reviewed sample; effect on EXP-03 forecast error | Layer 2 rule set |
| EXP-03 | L3 | Which price forecaster is best per crop/horizon? | Naive, seasonal naive, moving average, ETS, SARIMA(X), XGBoost, LightGBM, Chronos-2 (zero-shot), optional A3T-GCN | MAE, MASE, sMAPE, directional accuracy, coverage, runtime | Model promotion |
| EXP-04 | L3 | Are forecast ranges trustworthy at 30–120 days? | Quantile XGBoost, conformal intervals around any point model, Chronos-2 native quantiles, ETS intervals | Interval coverage (80/95%), pinball loss/CRPS, interval width | OI-02 contract; "no reliable signal" rule |
| EXP-05 | L3 | Does a better forecast give better decisions? | Same decision rule fed by each forecaster | Realised net ₹/q in backtest (EXP-07/10) | Whether accuracy gains matter |
| EXP-06 | L3 | Is the storage-risk rule sensible? | STORAGE-DEMO-0.1; ambient-RH-only rule; moisture-conversion (isotherm) rule; temperature-only rule | Agreement with one-time moisture measurements, category stability, sensitivity to thresholds | Freeze or revise storage policy |
| EXP-07 | L3 | When should held wheat / basmati be sold? | Sell at harvest; fixed hold to month M; forecast-threshold rule; forecast-with-uncertainty rule; perfect-foresight upper bound | Realised net ₹/q after storage loss and money cost; downside risk (5th percentile); regret vs perfect foresight | Hold/sell advisory (OI-01) |
| EXP-08 | L3 | Is the optimizer correct and fast enough? | SCIP, HiGHS, CP-SAT, greedy heuristic; brute-force oracle on tiny cases | Exact agreement with oracle on full objective sequence; solve/build/validate time; memory vs size | Production solver choice |
| EXP-09 | L3 | Which fairness/objective formulation? | Lexicographic (current); weighted sum; max-min only; leximin; proportional; utilitarian | Price of fairness (₹ lost), minimum farmer fulfilment, Gini of fulfilment, explanation clarity | Confirm or change objective hierarchy |
| EXP-10 | System | Does KisanNiti beat simple practice over past seasons? | Baselines: nearest mandi at harvest; MSP path; best-single-mandi rule; KisanNiti full pipeline | Net ₹/q per farmer, fairness, % lots sold in safe storage state; per-season breakdown | Paper headline result; value claim (OI-17) |
| EXP-11 | L4 | Is approval safe under concurrency, and usable? | Lock strategies (row lock vs serialisable); UI variants | Zero double-reservation in stress tests; task time, errors, SUS usability score (small user test) | Layer 4 implementation |
| EXP-12 | L5 | Do farmers understand the messages? | Punjabi/Hindi/English templates; with/without numbers in words | Comprehension quiz accuracy (with consent), delivery success, reply rate | Template wording; launch languages |
| EXP-13 | HW | Is the sensor node reliable? | Probe placements (in-grain vs ambient); buffer settings | Calibration offset, uptime %, data loss in Wi-Fi/power outages, settling time | Firmware/buffer settings; STORAGE policy settling time |
| EXP-14 | System | Does it scale? | Normal (5/1,000/3,000) and stress (50/10,000/30,000) fixtures | Latency, throughput, DB size, cost/month | Hosting choice (OI-12) |

## 2. Detail per layer

### Layer 1 — data (EXP-01)

- **Why these sources:** official AGMARKNET data via data.gov.in is the authoritative public source; CEDA republishes it for research with long history. Third-party sites (e.g. aggregators) are excluded as primary sources because their terms and processing are unknown.
- **Series:** wheat (Punjab mandis, especially Patiala district: Rajpura, Patiala, Samana, Patran, Nabha and others) and basmati paddy varieties (1121, 1509, 1718, 1885 and others as named). Include Punjab + Haryana markets so regional pooling can be tested.
- **Measure:** first/last date, days with modal price, longest gap, arrivals availability, duplicate/revised rows, variety-label variants, publication delay (record first-seen time for 30 days).
- **Decision rule:** a series is "forecastable" for a horizon only if it has ≥ 3 seasons with ≥ 60% of trading days observed in season (starting value — revise after seeing the data and record why).

### Layer 2 — cleaning (EXP-02)

- **Why compare:** mandi data contains typing errors and genuine spikes. Deleting spikes hides real opportunity; keeping errors breaks models.
- **Ground truth:** hand-review a stratified random sample (e.g. 500 rows, including all flagged and a random set of unflagged rows). Label each as error / genuine / unclear, with reasons.
- **Metrics:** precision and recall for "error"; downstream change in EXP-03 MAE when each cleaned dataset is used.
- **Decision:** choose the simplest rule set whose downstream error is not significantly worse than the best.

### Layer 3a — forecasting (EXP-03/04/05)

- **Why these candidates:**
  - **Naive/seasonal naive/moving average:** mandatory baselines; prices are sticky, so many models fail to beat them.
  - **ETS/SARIMA:** classic statistical methods that work with little data and give interpretable intervals.
  - **XGBoost/LightGBM:** strong tabular learners using lags, calendar, arrivals, MSP announcements and festival features.
  - **Chronos-2:** a pretrained time-series foundation model that can forecast without training. Check overlap between its pretraining data and our test period.
  - **A3T-GCN (optional):** tests whether linking markets helps, only if enough aligned multi-market data exists.
- **Horizons:** 1, 7, 14, 30, 60, 90, 120 days (the long horizons are needed for the hold/sell advice).
- **Protocol:** expanding-window rolling origin, weekly origins in each season; the final season is held out.
- **Metrics:** MAE (₹/q), MASE vs seasonal naive, sMAPE, direction-of-change accuracy, 80/95% interval coverage, pinball loss, training and inference time, and memory.
- **Significance:** Diebold–Mariano test of each model against the best baseline per horizon.
- **Promotion rule:** a model is promoted for a crop/horizon only if it beats the best baseline with p < 0.05 **and** its intervals are within ±5 points of the nominal coverage. Otherwise the baseline is used, and the advice says the signal is weak.

### Layer 3b — storage risk (EXP-06)

- **Why compare:** we chose in-grain water activity because mould limits are crop-independent and do not need paywalled isotherm constants. Alternatives test whether simpler or more physical rules do better.
- **Data:** bench node in a labelled 1 kg wheat sample at three controlled conditions (dry, room, humidified), plus one-time moisture-meter readings; Patiala seasonal weather for simulation.
- **Metrics:** agreement of category with measured moisture bands (wheat ≤ 13% / paddy ≤ 14% = safe), number of category flips per week, sensitivity of categories to ±2 RH points and ±2 °C sensor error.
- **Decision:** keep STORAGE-DEMO-0.1 unless an alternative agrees better with measurements *and* is no less stable.

### Layer 3c — hold-or-sell advice (EXP-07, new; see OI-01)

- **Why:** for held wheat and basmati, *when* to sell matters more than *where*.
- **Policies:**
  1. Sell at harvest.
  2. Hold to a fixed month.
  3. Sell when the forecast gain is below a threshold.
  4. Sell when the lower forecast bound beats today's price after costs.
  5. Perfect foresight, as an upper bound only.
- **Costs included:** storage loss rate (scenario values from the literature, labelled assumptions), money cost of waiting (interest rate scenario), and the storage-risk category.
- **Metrics:** mean and 5th-percentile realised net ₹/q per season; regret against perfect foresight; number of seasons where the policy loses to "sell at harvest".
- **Scenarios:** normal year; price-spike year (e.g. 2022 wheat); glut/low-price year; rain-damage year; monsoon storage-loss stress.

### Layer 3d — optimizer and fairness (EXP-08/09)

**Mathematics (for the paper):**

- **Decision variable:** integer y[l,m,d] ≥ 0. The quantity sent is x = Δ·y, where Δ is the scheduling increment.
- **Constraints:**
  - Per lot: Σ x ≤ Q_plan[l] (partial mode) or = Q_plan[l] (full mode).
  - Only admissible (lot, mandi, date) choices.
  - Transport cost uses fixed + variable terms with tight bounds.
  - Market-load limit applies only in STRICT mode.
- **Objectives, solved in order:** each stage is optimised, then fixed as a constraint for the next.
  1. Maximise total allocation.
  2. Leximin of farmer fulfilment r_f = S_f/P_f.
  3. Prefer HIGH-risk lots, then MEDIUM, on earliest dates.
  4. Maximise expected net return.
  5. Deterministic tie-break.
- **Leximin MILP:** for each k, maximise k·θ_k − Σ_f u[f,k] with u[f,k] ≥ θ_k − r_f and u ≥ 0. This equals the sum of the k smallest ratios.

**Experiments:**

- **Why these solvers:**
  - **SCIP:** open-source MILP; the candidate to deploy.
  - **HiGHS:** a second open-source MILP solver.
  - **CP-SAT:** an exact integer solver with a different method.
  - **Greedy heuristic:** shows how much optimality is worth.
  - **Gurobi (optional):** a reference only if an academic licence is available.
- **Correctness:** exhaustive enumeration on tiny cases (≤ 4 lots, ≤ 3 mandis, ≤ 3 dates). Every proven-optimal solver result must match the oracle's complete objective tuple.
- **Scaling grid:** farmers {10, 50, 200, 1,000} × mandis {3, 10, 25} × dates {3, 7, 14} × admissible density {20%, 60%, 100%}. Measure build, solve and validate time, memory, and timeouts at 60 s and 300 s.
- **Formulation comparison (EXP-09):** run the same instances with weighted-sum objectives (several weight sets), max-min only, proportional fairness and pure net-return maximisation. Show:
  - **price of fairness** = net return lost compared with pure maximisation;
  - minimum fulfilment, Gini, and how often weighted sums pick unfair plans.
- **Decision:** choose the solver that is correct on 100% of oracle cases and fastest at the realistic size found in EXP-14/FPO data. If exact leximin is too slow above some size, test the equivalent progressive-bottleneck method before considering approximations (an approximation must be labelled).

### Layer 4 — review and approval (EXP-11)

- **Correctness:**
  - Randomised concurrent approve/sell/release sequences against real PostgreSQL; the invariant is never violated.
  - Duplicate and lost-response cases (L4T08–L4T11).
- **Usability:**
  - 3–5 participants (students or FPO staff, with consent) complete review, edit, approve and release tasks.
  - Measure time, errors and System Usability Scale (SUS). Compare two UI variants only if time allows.

### Layer 5 — communication and outcomes (EXP-12)

- Template comprehension in Punjabi, Hindi and English with consenting participants. Show a message, then ask about quantity, mandi and date.
- Delivery success and timing on the WhatsApp test number.
- J-form-based sale entry accuracy, comparing duplicate detection by J-form number with manual entry.

### Hardware (EXP-13)

- **Salt calibration:** saturated NaCl, about 75% RH, for 24 h.
- **Settling-time test** after insertion into grain.
- **Outage tests:** 48 h runs with planned Wi-Fi and power cuts; count lost, duplicated and late samples, and check timestamps.

### Whole system (EXP-10, EXP-14)

- **Historical simulation:** replay past seasons (e.g. 2016–2025, as available), giving the pipeline only the data known at each date.
- **Synthetic farmers:** the FPO fixture farmers get lots sized from published Punjab averages (a labelled assumption).
- **Comparison:** KisanNiti vs the baselines in the summary table.
- **Results per season and per scenario:** normal year, spike year, glut year, rain damage, monsoon storage stress, data outage (missing week), sensor offline.
- **Load:** normal and stress fixtures; record cost per month for the chosen hosting.

## 3. Threats to validity (to report in the paper)

- Mandi modal price ≠ the price a specific farmer receives.
- Synthetic farmer lots and costs.
- The historical simulation cannot capture farmer behaviour or market reaction to advice.
- A single sensor node.
- Possible overlap between Chronos-2 pretraining data and our test data.
- Patiala-specific findings may not generalise.
