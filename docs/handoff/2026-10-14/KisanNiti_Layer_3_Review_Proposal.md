# KisanNiti — Layer 3 Forecasting and Allocation Proposal v0.1

> **14 October 2026 integration update:** Start with [Integrated Layer 1–3 handoff](KisanNiti_Layers_1_to_3_Handoff.md) and its acceptance plan. They consolidate this document and explicitly supersede obsolete hardware/location assumptions. Current hardware: ESP32-WROOM-32 + protected I2C SHT30 probe, with no manual moisture-meter requirement. Older hardware proposals below are retained only as history. No implementation/test completion is implied.

Date: 14 October 2026
Status: Design proposal for review. No model training, solver implementation or production benchmark has been performed in this work.
Basis: repository consolidated architecture sections 17–44 and current chat Layer 1 v1.2 / Layer 2 v0.2. The repository implementation blueprint is not changed by this document.

## 1. Purpose and boundaries
Cross-layer acceptance requirement: component, contract, adjacent-layer integration and end-to-end checks follow [Integration, Testing and Hardware Guide](KisanNiti_Integration_Testing_and_Hardware_Guide.md). Model evaluation, solver verification and real hardware integration have separate evidence requirements.
Layer 3 turns usable information into an explained proposal: which selected lots, how much, which candidate mandis, and which dates.
It contains two separate capabilities: forecasting provides estimated future prices; optimization selects an allocation under agreed constraints and priorities. Offline training is a separate lifecycle job, not a step repeated on each planning request.
Layer 2 owns observational cleaning, validation and readiness. Model-specific feature transformations consume its datasets and follow its leakage/lineage rules. Layer 4 owns manager approval; Layer 5 records communication and actual outcomes. A Layer 3 proposal does not reserve stock, dispatch produce or record a sale.

## 2. Classification of decisions
Carried-forward DESIGN DECISIONS: one FPO and commodity per plan; manager-selected lots and candidate markets; exact quantities and visible remainder; model-independent PriceForecast; solver-independent optimizer; per-farmer true leximin fairness; separate plan validation; versioned evidence and policies; training separated from inference.
Carried-forward preferred objective hierarchy: technical validity and economic acceptance, then allocation, fairness, storage urgency, expected net selling return and deterministic tie-break. Real-FPO acceptance and economic/storage guardrails remain unresolved.
New ENGINEERING LEANS for review: a seven-day demonstration horizon, partial mode as the demonstration default, XGBoost as the first trainable price model to evaluate, and SCIP via PySCIPOpt as the first MILP prototype backend. These are not claims that these choices win benchmarks or are frozen for production.
Chronos-2 remains the first pretrained benchmark and A3T-GCN remains an optional graph-model experiment. Neither is adopted as the production winner.

## 3. Planning request and demonstration scope
Each request identifies FPO, one commodity, selected lots with their participating quantities, candidate mandis, target dates and allocation/policy versions.
For each lot: 0 < participating quantity <= available snapshot. The manager may submit less than all available stock.
Proposed development profile: start with the next seven calendar dates, with only usable/allowed dates admitted; do not assume every mandi opens every day. Same-day dispatch is not part of this initial demo profile, pending actual operational cutoff rules.
Use the Layer 1 fixture's 20 candidate mandis for each FPO where compatible, without a production platform limit of 20. Candidate markets are visible and manager-controlled. Any shortlist or excluded choice has an explicit reason; no global all-India optimality claim over a subset.
National historical data can train/evaluate models while each planning problem covers only one FPO's selected commodity, lots, mandis and dates.

## 4. Forecasting target and candidates
Primary target: future reported modal mandi price with declared commodity, market and variety/grade applicability, explicit currency/unit and target date. It is not a guaranteed buyer offer or an individual lot's achieved price.
One output contract supports every candidate: market/commodity, applicability, target date, point forecast, generated-at time, input cutoff and versions, model version, status, and optional interval/quantile metadata. Missing or invalid forecasts remain unavailable; no fabricated zero fallback.

Evaluation sequence:
1. Last usable observed price and a past-only moving-average baseline, with explicit observation age and window definition.
2. XGBoost regression as the first trainable benchmark. Initial candidate features: lagged modal prices, past rolling summaries, observation-age/missingness indicators, known calendar features, and historical arrivals where usable. Future actual arrivals/weather/prices are not available features.
3. Chronos-2 as a pretrained benchmark, initially using documented inference with real historical context. Fine-tuning is a separate experiment only if results justify it.
4. A3T-GCN only after aligned multi-market history and a defensible graph are available. Define nodes/edges and build learned relationships using training data only. A trading relationship is not established by geographic proximity.

For seven-date forecasting, the trainable pipeline must explicitly handle the forecast horizon. A proposed initial XGBoost experiment uses horizon-specific targets/features known at the forecast origin; it must not use actual day-1 prices as inputs when generating day-7 forecasts at the original request time. Compare alternatives only when useful.
Pool compatible series only through an explicit model design and features. There is no assertion that one model trained on all mixed commodity prices without context is valid.
The model does not use FPO-specific stock to change a general mandi observation forecast without a separately justified causal/predictive design. Stock is principally an optimizer input.

## 5. Forecasting evaluation and promotion
Use chronological rolling-origin validation, an untouched later test period, and the same target-date/horizon definitions for comparisons. All learned preprocessing is fitted within the applicable training period. Preserve first-known/revised data limitations and investigate pretrained-data overlap.
Report MAE in an explicit common price unit, error by commodity/market and forecast horizon, coverage/abstentions, performance during genuine price spikes, inference latency and resource requirements. Avoid judging different-coverage models using a single aggregate error without stating the coverage difference.
Evaluate any reported prediction intervals for coverage/calibration. A model quantile output is not a guarantee of that accuracy in mandi data. Initial optimization uses the declared point forecast; uncertainty is visible context until a risk-aware objective is explicitly designed.
Promotion requires improvement or a defensible accuracy/cost/coverage tradeoff versus simple baselines. Preserve a model/dataset/feature manifest, evaluation report and rollback version. No fabricated accuracy percentage or required universal minimum history length.

## 6. Training and update lifecycle
- Train/evaluate candidates offline after the usable historical dataset is prepared. Track exactly what was trained versus used pretrained.
- New successful market batches update dependent features/forecasts using the current promoted model. Combine changes and skip unchanged input versions.
- Training is not triggered by each price, sensor event, sale or manager request.
- For initial development, retraining is an explicit offline evaluation run. Once model behavior and data volume are measured, select a scheduled review/retraining cadence; never silently promote an unevaluated candidate because a timer fired.
- Sensor updates affect supported storage assessments; inventory events affect availability; quote changes affect costs. These do not automatically retrain the price model.
- A manager's plan/replan request runs the optimizer using usable versioned inputs. New evidence can flag a prior revision as changed; it cannot silently rewrite an approved plan.

## 7. Optimizer input and decision
Inputs: participating lots/ownership and available snapshot, candidate markets/dates, applicable forecasts, routes, transport-cost model, supported storage assessments, economic/allocation/fairness/storage/market-load policies, and quantity/numeric conventions.
The application obtains provider information and prepares a validated frozen input before calling the engine. The engine makes no network calls and performs no database writes.

Decision x[l,m,d] means proposed quantity of lot l allocated to mandi m on date d.
It is not an actual sale or stock reservation. A lot can have several assignments across mandis/dates without creating a physical split of its identity.
Quantities respect the declared lot scheduling increment: x = increment * nonnegative integer. The production increment remains an explicit business decision; canonical kg does not imply a 1 kg planning increment. Development tests can exercise multiple explicitly synthetic increments without adopting them as production rules.

Partial mode: total allocated from a lot is at most its participating quantity. Show participating, allocated and unallocated amounts separately. Proposed demo default is partial mode.
Full mode: every selected participating quantity must be allocated. Detect incompatible quantities/increments early; if the required plan is not possible under the declared constraints, explain the result rather than secretly switching modes.

## 8. Technical constraints and missing evidence
- Correct FPO, commodity and authorized selected lots.
- No allocation beyond participating quantities or available snapshot; current availability is rechecked at approval.
- Only allowed dates and candidate mandis, supported variety/grade applicability, valid forecasts, and usable required transport economics.
- Travel/date feasibility must use the declared origin, travel duration and operational date semantics; unknown cutoffs do not justify a claimed executable same-day journey.
- Vehicle/load limits only where supported by the transport contract. Do not invent unlimited vehicle availability or combine unlike quotes.
- Missing evidence removes the affected technical choice with an explicit reason; it is not proof of business infeasibility. Malformed mixed-FPO input is invalid as a whole.
- Storage UNKNOWN does not itself forbid allocation; no fabricated urgency or deterioration trajectory.
- Historical arrivals are not official physical capacity. Proposed demo MarketLoadPolicy is DISABLED until a clearly described advisory context is prepared. ADVISORY may display context but does not constrain feasibility or penalize the objective. STRICT requires validated provenance, calibration and a deliberate production decision.

## 9. Transport economics
Use quote-derived economics with explicit origin/destination/load/vehicle/charge applicability. The modeled transport charge must correspond to the proposed load and any declared trip grouping.
A full-trip quote is not a per-kilogram rate and must not be charged in full to every farmer sharing a trip. Before enabling shared-trip optimization, freeze the grouping/capacity/availability and cost-accounting contract. V1 is not a fleet dispatch or vehicle-routing product by default.
Development can use explicitly synthetic dedicated-trip/load scenarios to exercise correct quote application. This does not establish a universal production tariff.
Gross expected receipt = sum of applicable predicted price multiplied by allocated quantity.
Expected net selling return = gross expected receipt minus modeled transport cost.
This is not full farming profit: other costs are not implicitly included. Show excluded/unknown charges, and keep actual expense separate from an estimate.

## 10. What best means: existing objective hierarchy
First satisfy technical/domain validity and the declared economic-acceptance policy. Within that allowed region:
1. Maximize useful allocated quantity in partial mode.
2. Apply true leximin farmer fulfilment fairness.
3. Apply supported storage urgency: HIGH total/timing, then MEDIUM total/timing, preserving higher objectives. LOW has no extra early-sale pressure and UNKNOWN no invented rank.
4. Maximize expected net selling return.
5. Resolve remaining ties deterministically.

Lexicographic priorities mean each later objective preserves the earlier achieved optima. Do not blend rupees, fairness and urgency into arbitrary percentage weights.
This design does not simply maximize financial return. Allocation/fairness/storage can constrain the final money outcome. The actual FPO must accept these priorities, and an economic guardrail must prevent unacceptable sacrifices before production use.
In full mode, allocation totals and farmer fulfilment are already fixed; skip redundant stages. Storage timing and financial choices may still matter.

Fairness illustration: farmer A participates with 100 kg and B with 300 kg. If only 200 kg can be allocated, all choices are equally admissible/economic and the increment permits it, 50 kg for A plus 150 kg for B gives each 50% fulfilment. Equal kilograms (100 each) would give 100% and about 33%. Multiple lots do not give a farmer extra fairness weight. This illustrates allocation fairness, not equal income or proven optimality in every constrained case.
Full true leximin improves the worst fulfilment ratio first, then the next worst and so on; only maximizing the single smallest ratio is insufficient.

## 11. Model and solver implementation path
Forecasting ML and the allocation solver are separate. The optimizer is a mathematical constrained problem, not a neural network that must learn allocations from historical sale labels.
Retain the architecture's MILP-first prototype direction. MILP means mixed-integer linear programming: express discrete allocation choices, constraints and objective stages mathematically and use an existing solver.
Proposed first prototype: SCIP through PySCIPOpt behind OptimizationEngine. SCIP is an existing mixed-integer optimization library; we build KisanNiti's formulation, interfaces and independent checks rather than inventing a numerical solver.
Benchmark a second appropriate backend such as HiGHS under identical business semantics. CP-SAT remains a possible discrete alternative; Gurobi is a reference option subject to licensing. These are engineering comparisons, not compulsory production dependencies.
True leximin and staged priorities can require multiple solves and a sizable formulation. Benchmark realistic one-FPO cases, not just small examples; do not simplify the fairness semantics invisibly to claim speed.
Exact domain quantities/money must be reconstructed and independently checked after solver execution. Numerical tolerances and integer scaling are versioned implementation settings, never permission to over-allocate stock.

## 12. Result, explanations and failure states
Return a candidate and its proof/status separately from a saved plan revision:
- Allocations: lot, farmer, mandi, date, exact quantity.
- Participating/allocated/unallocated quantities and reasons for missing choices.
- Predicted prices, gross, modeled transport cost and expected net selling return.
- Farmer fulfilment fractions and supported storage outcomes.
- Selected search boundary, evidence/model/policy versions, generation time and data age.
- Whether feasibility and each objective stage were established, termination reason, runtime and any remaining optimality gap information with proper stage semantics.

Distinguish invalid input, insufficient usable evidence, proven infeasibility, zero useful allocation, feasible-but-not-fully-proven result, and technical timeout/failure. Timeout alone proves neither infeasibility nor optimality.
Whether an incompletely optimized but valid candidate can become an approvable proposal is a later explicit product policy. It may be shown as a clearly labelled diagnostic candidate during development without implying approval eligibility.
Explanations derive from stored inputs, constraints and objective results: show cost differences, excluded choices, fulfilment and remainder. Claim that an alternative is dominated or impossible only when supported by a check or solve; do not invent causal explanations from the output alone.
All candidates pass the independent Layer 2 PlanValidator before promotion to an evaluated revision. Layer 4 can accept or change decisions; changed decisions get a new revision and revalidation. The proposal itself reserves no stock.

## 13. Simple cost example (synthetic, not a forecast result)
Consider 1,000 kg, the same sale date, two admissible mandis, equally satisfied higher priorities and quotes explicitly covering the full load:
- Mandi A: predicted INR 24/kg; gross INR 24,000; applicable transport INR 1,000; expected net selling return INR 23,000.
- Mandi B: predicted INR 25/kg; gross INR 25,000; applicable transport INR 3,000; expected net selling return INR 22,000.
Among these two choices, A gives the higher expected net selling return despite B's higher predicted market price. This arithmetic is an illustration, not a solved production allocation or guaranteed sale outcome.

## 14. Validation and completion evidence
Forecasting: rolling time-based evaluation, simple baselines, honest data coverage, fixed evaluation targets, leakage checks, optional interval calibration, versioned artifacts and measured inference cost.
Optimization: enumerate every feasible assignment for tiny cases and compare the entire priority sequence with solver results; compare backends where practical; independently recompute inventory, economics and fairness.
Include cases for multiple lots per farmer, differing participating quantities, full/partial modes, valid remainders, missing routes/forecasts, UNKNOWN storage, incompatible increments, shared-cost handling where enabled, multiple optimal solutions, timeouts and zero allocations.
Integration: Layer 2 usable snapshot -> forecast contract -> validated optimizer input -> structured result -> independent plan check -> immutable evaluated revision for Layer 4. Add later approval/partial-sale/replan scenarios to the existing end-to-end acceptance suite.
Performance: report runtime by number of farmers, lots, admissible choices, dates and objective stages. The archive can contain millions of rows without each solve scanning those millions of rows. Do not promise a latency target before measuring.

## 15. Decisions needed before this layer can be production-frozen
1. Real allocation increments, full/partial expectations and operating/dispatch dates.
2. Economic acceptance and the guardrail for storage/fairness versus financial outcome.
3. Actual transport quote/grouping/vehicle availability semantics, plus any farmer-level cost attribution needed by policy.
4. Validated storage thresholds and acceptance of the fairness/priority hierarchy.
5. Market-load enforcement/calibration if enabled.
6. Supported forecast series/horizon, evaluation results and chosen model/update policy.
7. Solver benchmark results and policy on feasible-but-not-proven candidates.
8. Production freshness/approval rules at the Layer 4 boundary.

These unresolved business/calibration questions do not imply an impossible software design. Development fixtures can demonstrate the mechanics with explicit policies; they cannot certify those policies for a real FPO.
Before implementing/training any model, present its inputs, purpose, evaluation plan and evidence as requested by the user. This document is that design proposal, not evidence of completed training.

## Sources
- Existing repository consolidated architecture, sections 17–44: model candidates, MILP-first formulation, fairness semantics, objective hierarchy and constraints.
- Chronos-2 publisher model card: https://huggingface.co/amazon/chronos-2
- XGBoost official parameters: https://xgboost.readthedocs.io/en/stable/parameter.html
- SCIP official project and Python interface entry: https://www.scipopt.org/
- Hyndman and Athanasopoulos, rolling-origin time series evaluation: https://otexts.com/fpp3/tscv.html
- No external source establishes our seven-day demo horizon, economic policy, fairness acceptance or final solver/model choice. Those are project proposals or decisions requiring evidence.
