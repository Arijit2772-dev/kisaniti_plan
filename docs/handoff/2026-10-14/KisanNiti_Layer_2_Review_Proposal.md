# KisanNiti — Layer 2 Cleaning, Validation and Readiness Proposal v0.2

> **14 October 2026 integration update:** Start with [Integrated Layer 1–3 handoff](KisanNiti_Layers_1_to_3_Handoff.md) and its acceptance plan. They consolidate this document and explicitly supersede obsolete hardware/location assumptions. Current hardware: ESP32-WROOM-32 + protected I2C SHT30 probe, with no manual moisture-meter requirement. Older hardware proposals below are retained only as history. No implementation/test completion is implied.

Date: 14 October 2026
Status: Proposal for user review, not a frozen implementation blueprint or implemented feature.
Basis: Final Layer 1 Design v1.2 and the repository's consolidated architecture, especially quantity representation, forecasting, technical admissibility, validation and approval freshness.
The Layer 1 baseline remains unchanged. This document makes new recommendations explicit and does not silently close real-FPO unknowns or authorize product-code implementation.

Revision note: v0.2 explicitly locates cleaning in Layer 2, adds the evidence and evaluation basis, and revises v0.1's automatic withholding of statistically unusual prices. A statistical flag alone now does not exclude or replace an otherwise consistent observation. The previous 30/15/6-MAD settings remain an unvalidated experimental candidate, not a production rule.

## 1. Purpose
Cross-layer acceptance requirement: component, contract, adjacent-layer integration and end-to-end checks follow [Integration, Testing and Hardware Guide](KisanNiti_Integration_Testing_and_Hardware_Guide.md). A rule is not implemented-and-verified merely because it appears in this proposal.
Layer 2 answers: what does this input mean, is it internally consistent, and is it usable for this particular operation at this time?
Validation cannot prove that a reported quantity was physically weighed correctly, that a published price was truthful, or that an ambient sensor reading proves crop quality. Source evidence and real operational verification remain distinct from software checks.

Selected conceptual flow:
permitted source record -> explicit parsing and normalization -> identity/unit/consistency checks -> quality findings -> purpose-specific readiness -> versioned usable dataset or review queue.

Validation is a shared capability throughout the application, not a single one-time filter. Run it at ingestion, model-input preparation, optimizer-input preparation, plan-output checking, and approval. Each stage has different responsibilities.

### Where cleaning and models live
- Layer 1 collects and preserves source information; its boundary checks establish whether a payload can be safely parsed and stored.
- Layer 2A profiles and cleans: explicit parsing, ID/unit normalization, duplicate/revision reconciliation and identification of missing/invalid/suspicious fields. Original source records remain preserved where permitted.
- Layer 2B validates: checks the cleaned representation and business relationships against versioned rules.
- Layer 2C assesses readiness for a named downstream use and publishes the selected dataset/version with issue flags and coverage.
- Model-specific feature preparation is part of the forecasting pipeline consuming Layer 2 output. Learned transformations are fitted only on the applicable training period, saved with the model pipeline, and reused at inference.
- Forecasting feeds Layer 3 planning before optimization. Offline training/evaluation is a separate job using Layer 2 datasets; neither cleaning nor receiving a record automatically starts training. Forecasting and optimization have different responsibilities.
- Layers 4 and 5 perform manager decisions, communication and actual-outcome recording; shared validators are reused at their boundaries.

No cleaning model, imputation model, forecasting model or GNN has been built/trained by this design work. Initial cleaning is deterministic rules plus optional statistical flags. A learned anomaly detector or model-based imputer would be a separately described and evaluated proposal before implementation, with its inputs, purpose, evidence, failure behavior and cost explained to the user first.

## 2. Decision classifications
- DESIGN DECISION carried forward: exact inventory/money, tenant isolation, auditable corrections, explicit unavailable/UNKNOWN states, independent PlanValidator, transactional approval recheck and no provider calls inside pure validators.
- DESIGN DECISION proposed for review: the issue/readiness contract, per-source rules, correction workflow, and integration/acceptance criteria below.
- ENGINEERING LEAN proposed for review: kilogram/INR-per-kilogram normalization, Pydantic for boundary schemas, DuckDB for historical batch checks, the initial anomaly-screening rule and a development sensor-age setting.
- UNKNOWN for production: actual FPO verification/override authority, minimum allocation increment, economic acceptance, quote validity where not supplied, approval freshness actions and scientific crop-specific storage thresholds. Interfaces can require these policies; an absent production policy is not silently replaced by a demo assumption.

## 3. Report quality, freshness and readiness separately
Do not compress all concerns into one opaque quality score.

Record quality:
- PASS: checks for the declared record contract passed. This is not certification of physical truth.
- REVIEW: ambiguous, suspicious or conflicting evidence needs resolution for the affected use.
- FAIL: an explicit rule failed, such as missing identity, an invalid unit or an impossible value.

Freshness facts are separate: observation age, source publication time when known, last successful collection, explicitly supplied expiry, newer/corrected version available, and whether freshness could be assessed. Age alone does not invalidate history.

Readiness is computed for a named purpose and policy version:
- READY: all required inputs passed applicable rules.
- READY_WITH_WARNINGS: remaining issues are explicitly allowed as non-blocking by that policy.
- BLOCKED: a required input/rule is unsatisfied; include the exact affected record, series, choice or operation.

Every finding has a stable code, plain-language explanation, field/record reference, blocking scope, evidence reference and suggested corrective action. Record validator version, policy version, input versions and evaluated-at time. Readiness can differ for archival storage, training, inference, planning and approval.

Example: missing arrivals may permit a modal-price-only model but block a model that requires arrivals. Missing storage assessment yields UNKNOWN with an explanation; it does not itself forbid a selling choice under the current architecture.

## 4. Common normalization rules
1. Retain permitted raw records unchanged; create linked normalized records and transformations separately.
2. Use canonical IDs and versioned mapping tables. Trim irrelevant whitespace and normalize known formats. Fuzzy spelling matches are suggestions, not automatic merges of farmers, markets, commodities or varieties.
3. Proposed canonical mass unit: kg. Preserve original value/unit and convert only verified mass units, such as 1 quintal = 100 kg and 1 metric tonne = 1,000 kg. Bags/crates require a recorded mass conversion for that context; no assumed universal bag weight.
4. Proposed canonical market-price rate: INR/kg, preserving the original INR/quintal or other source representation. Do not treat a total invoice amount as a per-unit rate. Foreign/unknown currency requires explicit handling; no silent FX conversion.
5. Exact decimal arithmetic for inventory, money and rates. Decimal storage scale, business rounding and maximum supported magnitudes must be specified before implementation; inputs requiring unsupported precision are flagged instead of silently rounded. Display rounding must not change inventory truth.
6. Canonical kg is distinct from the optimizer's minimum allocation increment and internal solver scale. A recorded actual sale is not forced onto a planning increment.
7. Parse dates using the source's declared convention. Preserve market business dates; use UTC for technical timestamps and Asia/Kolkata for Indian business scheduling. Ambiguous dates and undocumented timezones require resolution.
8. Require finite numeric values. Missing, zero, unavailable and parse failure are distinct. Never silently convert a missing number to zero.
9. Duplicate detection is source-aware. Prefer source identity/revision; otherwise use a documented complete business key and content fingerprint. Different grades/varieties or legitimately separate reports must not collapse together.
10. A proven correction creates a linked version. Conflicting values without proof of supersession enter review; latest receipt time alone does not prove which value is right.
11. Reconcile overlapping sources with explicit provenance and priority. Do not count an AGMARKNET observation republished by another provider as a second independent observation or average conflicting providers silently.
12. Keep synthetic data outside real training/evaluation and live operational readiness, with explicit test-mode authorization to use it in demonstrations.

## 5. Organization, inventory and sales checks
- The authenticated actor may access and change only the authorized FPO records. FPO, member, lot and location references must be consistent. Failed authorization never becomes a manager-overridable warning.
- One lot represents one commodity owned by one farmer member; multiple lots are supported.
- Initial lot quantity must be positive and exact, with a supported unit. Zero remaining inventory is a valid sold/exhausted state, not an invalid original lot.
- Store quantity evidence and verifier when supplied. The development intake-operator role from Layer 1 is a fixture assumption, not a validated real-FPO authority model.
- Require one unambiguous active placement for dispatch planning and keep placement history. Missing origin blocks the affected route-based choice rather than causing an invented origin.
- Reject duplicate mutation requests using stable event/request identities; preserve authorized corrections through adjustment/version history.
- Derive available quantity from authoritative stock, actual sold quantities, remaining active reservations and finalized adjustments. Do not trust a client-supplied remaining-stock total.
- No over-allocation or negative available stock. Two simultaneous approvals must not reserve the same stock twice.
- A linked sale consumes its corresponding reservation and records actual sold quantity through the agreed transaction path; it must not reduce availability twice.
- A real off-plan sale can require reservation reconciliation. Preserve the report and route the unresolved transaction through the later agreed workflow; do not automatically select someone else's reservation or pretend a failed inventory mutation succeeded.
- Do not apply the market-observation positive-price training rule blindly to actual transactions; disputed, free or incompletely priced real events need their own explicit recording semantics.

## 6. Market-observation checks
Required for a price-target record: resolvable market and commodity, observation date, interpretable modal price, price currency/denominator, source identity/provenance and declared variety/grade applicability (including missingness).

- Observed modal prices must be finite and positive to be used as ordinary price targets. Zero/negative source entries remain in the raw archive with an issue; they are not usable price targets without separate source semantics.
- Where supplied, minimum <= modal <= maximum. A contradictory range enters quarantine for the affected price record; missing optional minimum/maximum does not by itself invalidate a usable modal price.
- Evidence nuance: the indexed official AGMARKNET data-entry SOP describes strict ordering for its entry form. Our non-strict historical consistency check is an engineering choice, not a verbatim reproduction of that form rule. Equality cases require the relevant export/version semantics to be documented; do not claim publisher validation that has not been checked.
- Arrivals must be non-negative with units where supplied. Genuine zero is preserved. Missing arrivals are not zero and do not invalidate an otherwise usable price-only observation.
- Future-dated records cannot be historical observations as of the collection context; investigate source/date parsing. Forecasts are stored separately and can legitimately target future dates.
- Market, commodity, variety, grade and unit distinctions survive normalization. Missing variety is not silently labelled Common; Paddy(Common) as a commodity label does not establish a separately reported variety.
- Missing reporting days remain missing. Do not invent observations for closed markets, holidays or non-reporting dates.
- Show series coverage, gaps, source revisions and last observation. A national row total does not establish local series readiness.

Anomaly screening — ENGINEERING LEAN for development:
- Compare a new positive modal price with the preceding 30 usable observations of the same comparable series; require at least 15 previous observations.
- Flag if its absolute difference from the prior median exceeds 6 times the unscaled median absolute deviation (MAD). Use only preceding data and do not include the point under test in its own baseline.
- When MAD is zero or history is insufficient, report that this screen could not assess the point; do not divide by zero or invent a fixed percentage fallback.
- This is a provisional review heuristic, not proof of incorrect reporting or a validated agricultural threshold. Measure false alarms, seasonality effects and missed errors before adopting it in production.
- Revised default: an anomaly-score flag alone preserves the original price in the otherwise valid normalized/model-input data, with a non-blocking warning and review item under the development policy. It does not automatically delete, clip, replace or withhold the observation. Identity/unit ambiguity, contradictory source fields or other independent blocking evidence still quarantine the affected use.
- Confirmed erroneous records may be corrected from evidence or excluded from a named derived dataset, retaining original values and reasons. Resolve repeated issues by series/batch where evidence permits, rather than forcing identical per-row manual work.
- Do not clamp genuine price spikes into a normal-looking range. Reviewing a genuine spike preserves its original value and the review decision.
- The 30-observation window, 15-observation minimum and 6 unscaled MAD multiplier are our experimental settings; no cited source validates that combination for mandi prices. Evaluate them before production selection. NIST's separate modified-Z example uses a different scaled statistic and threshold; the two must not be misrepresented as identical.

## 7. Route and quote checks
Route result:
- Origin matches the lot's applicable placement and destination matches the selected market.
- Distance/time have explicit units, finite non-negative values, provider status, timestamp and provenance.
- An unavailable route is not a 0 km route. Zero may only represent a genuine provider-supported same-location case; otherwise investigate it.
- Coordinates and market-location mappings must be plausible and traced to their source. A numeric coordinate alone does not prove the correct mandi was mapped.
- Ordinary road-driving estimates are not proof of truck restrictions, accessibility or live road safety.

Transport quote:
- Validate currency, quoted amount/rate basis, origin/destination, vehicle/load applicability, included/excluded charges, source and explicit validity where supplied.
- Missing pricing or an incompatible load cannot supply a usable transport estimate. An explicitly evidenced zero-charge quote is distinguishable from a failed/missing quote.
- Missing charges remain unknown; do not silently mark toll/loading/unloading as zero. Surface the scope of the estimate.
- Do not silently invent expiry. A quote without validity metadata reports validity as unknown and needs the applicable later policy or authorized confirmation for operational use.
- Shared-trip quote use must follow the later costing contract. Validate once that it covers the proposed shared load; do not multiply a whole-truck quote per farmer or assume unlimited capacity.

## 8. Sensor checks and assessment boundary
- Verify device identity and authorized source. Preserve the device-to-zone assignment applicable at measurement time, not merely its current assignment.
- Parse temperature and humidity as finite values with explicit units; humidity must be within 0–100%. Validate the device's supported measurement range against its registered specification.
- Preserve measurement time and receipt time separately. Delayed replay stays historical; it does not become a fresh reading because it arrived now.
- Deduplicate using reliable message/device sequence identity where provided. Do not merge separate legitimate readings merely because their values are equal.
- Future/inconsistent/untrusted device timestamps produce a time-quality finding and cannot silently establish fresh measured evidence.
- Missing fields can be flagged per measurement channel; an assessment declares which channels it requires.
- Record device health separately from storage condition. A disconnected sensor does not mean healthy produce, and a working sensor does not prove healthy produce.

Development-only freshness recommendation: with Layer 1's five-minute sampling, show a delayed/stale indication after more than 15 minutes without a trustworthy usable measured reading. This is a configurable engineering demo setting, not a scientifically validated crop-risk threshold or a production approval TTL.
Without a valid, sufficiently current reading and a crop/context policy supported by evidence, storage assessment remains UNKNOWN. No invented HIGH/MEDIUM/LOW cutoffs or grain-moisture/spoilage claims.

## 9. Training and inference readiness
- Partition comparable series using declared commodity/market and variety/grade applicability. Cross-crop global models require explicit features and documented model design; do not concatenate unrelated prices into one apparent history.
- The selected model declares its required lookback, frequency, target, covariates, unit, missing-data support and minimum context. These belong to the model contract, not a universal claim that every crop needs the same number of rows.
- Produce dated coverage/missingness reports before training or inference. Unsupported series are visible as unsupported, not assigned fabricated predictions.
- Keep observed data separate from model-specific transformed features. If imputation is used, record the method and mask; never rewrite an imputed value as an observed source price.
- Training/validation/test splits follow time. Fit scalers, imputers, feature selection and learned graph relationships using training information only. No future-price interpolation into a past forecast context.
- Evaluation uses actual observed targets; imputed labels do not count as observed ground truth.
- Preserve observation time versus information-availability time; report limitations when historical release/revision times are missing. Pretrained model overlap/provenance also needs assessment.
- Sensor and FPO synthetic records cannot inflate real forecasting training or test counts.
- A GNN additionally needs aligned market histories and an explicit versioned graph. Validation does not infer trading links merely from nearby locations.

Forecast output checks:
- Known model/version and supported market/commodity/variety applicability.
- Target dates match the requested horizon; generation time and input cutoff are present and consistent.
- Price is finite, positive for this price-target contract, with explicit currency/denominator; optional bounds must be ordered when supplied.
- Invalid output is unavailable for the affected choice. Do not silently clamp it or use an untested fallback.
- Statistical accuracy is established through model evaluation, not by passing structural output checks.

## 10. Freshness and missingness decisions
No universal TTL applies to all four input groups.

| Situation | Proposed handling |
| --- | --- |
| Old market observation in a historical dataset | Keep it if valid; age is expected history |
| Latest market observation older than expected | Show observation age and collection status; model policy determines whether current inference is supported |
| New forecast or corrected source version exists | Flag changed evidence and refresh affected draft computations as appropriate; approved revisions remain immutable |
| Quote explicitly expired | Unusable for choices requiring that quote; obtain a valid replacement |
| Quote validity not supplied | Report unknown validity; require applicable policy/confirmation, not invented expiry |
| Missing or delayed sensor evidence | UNKNOWN storage assessment; warning, not automatic exclusion of the lot |
| Missing required forecast or route/cost input | Affected planning choice unavailable with reason; never substitute zero |
| Ambiguous variety/unit for required input | Review; affected use blocked until resolved |
| No usable candidates remain for a lot | Explain why; do not label missing evidence as proven market infeasibility |

Production approval warning/block thresholds, same-day cutoffs and model freshness limits need explicit policy decisions in their relevant layers. A development policy is versioned and labelled non-production. Choosing an arbitrary number here would not validate those business rules.

## 11. Three downstream validation gates
Gate A — OptimizationInputValidator:
- Checks FPO/commodity consistency, selected lots, exact available quantities/snapshot, candidate markets/dates, compatible forecasts, transport estimates, policy inputs and numeric preparation.
- A malformed request such as mixed FPOs is invalid as a whole. A missing route for one otherwise legitimate choice can make that choice unavailable without discarding every other choice.
- Required production policy configuration must exist; missing strict inputs do not become zero or infinity.

Gate B — independent PlanValidator:
- Applies the same deterministic hard rules to optimizer proposals, manager edits and manually constructed plans.
- Checks totals, stock limits, allocation increment, allowed choices/dates, full/partial semantics, declared strict market-load limits and economic hard requirements, and recomputed arithmetic.
- Does not equate historical arrivals with physical mandi capacity, invent minimum profit rules, or prove that a feasible plan is mathematically optimal.
- Uses frozen evidence and performs no provider calls. An edited decision produces a new revision and refreshed affected derived inputs.

Gate C — approval-time validation:
- Checks current actor authority, revision state, execution dates, current inventory/reservations and the active freshness policy.
- Obtains required external evidence before acquiring inventory locks; final inventory recheck/reservation/approval commit occurs atomically.
- No warning acknowledgment can override tenant isolation, invalid units, over-allocation or other invariant failures.
- Business-warning overrides require an explicit policy and audit; they are not a blanket bypass.

These gates enforce contracts belonging to later layers. This proposal does not silently select the optimizer's increment, economic policy, market-load mode or real-FPO approval authority.

## 12. Correction and review workflow
Proposed review screen columns: affected record/series, source, issue, impacted operation, age, allowed correction action and assigned responsibility.
Useful summaries: accepted/review/failed/duplicate/corrected counts, affected market/crop coverage, latest source dates and blocked planning choices.

Allowed actions according to authorization: correct an organization's own data with evidence; resolve an identity mapping; accept a plausible flagged observation with evidence; request/refetch a source correction; or exclude a record from a named derived dataset. Preserve the original source record in every case permitted by retention terms.
A manager does not rewrite an official historical price as if the publisher had reported it. A reviewed interpretation or derived correction is separate and traceable.
Store who acted, when, why, evidence, previous/new versions, and which downstream results became outdated.
Revalidation processes the changed records and their dependent series/features/plans; it does not unnecessarily scan the entire archive for each edit.
Automate deterministic corrections only when semantics are unambiguous. Business ambiguity goes to a human with the right authority.

## 13. Implementable technical approach
Proposed first implementation uses the existing FastAPI/PostgreSQL modular monolith:
- Pydantic schemas and explicit validators for API/device/provider boundaries, with strict validation after deliberate source parsing.
- PostgreSQL constraints, uniqueness and transactions as the final protection for identities, duplicate mutations and concurrent inventory updates. Cross-record stock invariants need the transaction/service logic, not only a single-column CHECK.
- DuckDB batch queries over Layer 1 Parquet for historical schema, unit, duplicate, range and coverage checks; publish versioned accepted/review datasets and manifests.
- Shared deterministic Python domain validators for business invariants and plans.
- Background jobs for historical batches and dependent revalidation; operational entry checks run at submission.

Use one versioned rule specification with conformance examples so API validation and bulk SQL checks agree. A change of rule version produces a new report/dataset, not invisible reinterpretation of old results.
No trained model is required for the core validation layer. Statistical screening is a limited review aid. GNN/Chronos/XGBoost belong to forecasting evaluation.
Dependency versions, schema precision, deployment settings and performance limits must be pinned/tested during the implementation blueprint and build. No claim of installed libraries or measured throughput is made here.

## 14. Example acceptance scenarios
1. 25 quintals with a verified unit maps exactly to 2,500 kg and retains the original value.
2. INR 2,400/quintal maps exactly to INR 24/kg; a missing price remains missing.
3. A repeated source record creates no duplicate normalized observation; a genuine correction retains both versions.
4. Same mandi/date but different varieties stays distinct; two source aliases for the same mandi resolve only through an approved mapping.
5. A negative modal price is withheld; a large positive spike with no independent error evidence is flagged while retained, and review can confirm it unchanged.
6. Missing arrivals do not prevent a compatible price-only model; they block a model that requires them without a supported missingness strategy.
7. A future model target is valid; a future-dated supposed historical observation requires investigation.
8. A failed route never becomes 0 km/zero cost, while a supported genuine same-location route can have zero distance.
9. An explicitly expired or incompatible-load quote cannot price that choice.
10. Relative humidity 140% fails; delayed old readings do not refresh current evidence; a moved device's history remains attached to its historical zone.
11. Sensor UNKNOWN does not by itself forbid planning.
12. A model transformation cannot use future observations in an earlier forecast, and imputed targets do not inflate evaluation counts.
13. A wrong-FPO request fails; current stock conflicts prevent double reservation during simultaneous approvals.
14. A manager-edited over-allocation fails the same PlanValidator as an optimizer output.
15. A duplicate sale event has no second inventory effect; linked sales consume the right reservation.
16. An interrupted batch resumes with matching counts and no duplicate publication. A changed mapping triggers only the necessary dependent revalidation.
17. A stale evidence warning cannot silently rewrite an approved revision or invent an unapproved automatic replan.

## 15. Outputs and next review boundary
Deliverables: data dictionary/normalization conventions, versioned validation rules, issue codes and repair actions, normalized datasets with provenance, coverage/readiness reports, review workflow, model-input contract requirements, shared input/plan/approval validation boundaries, and automated evidence for the scenarios above.

Items specifically proposed for user review: kg/INR-per-kg normalization; PASS/REVIEW/FAIL plus purpose-specific readiness; quarantine rather than deletion; provisional anomaly screening; the development-only 15-minute sensor-age indication; and shared deterministic checks throughout planning and approval.
This document proposes Layer 2. It does not finalize later-layer business policies, implement product code, claim 100% source truth or certify production readiness.

## 16. Evidence basis for cleaning
Cleaning is not adopted wholesale from one research paper. Each rule needs the kind of evidence appropriate to it:

| Rule family | Basis | What it does not prove |
| --- | --- | --- |
| Source fields, identifiers, date/unit conventions | Provider schema/data dictionary, documented exports and bounded source samples | That every reported record is physically true |
| Exact unit conversion, duplicate accounting, ownership and inventory consistency | Explicit units, identities, project invariants and executable examples | A valid physical weighing or an accurate source measurement |
| Outlier flagging | NIST discussion of MAD/modified Z-scores and outlier investigation; forecasting methodology on unusual observations | That an extreme mandi price is an error or a universal threshold works |
| Missing-value handling | Forecasting methodology: reason for missingness and model capability matter | That all gaps can be filled with the same method |
| Learned preprocessing and evaluation splits | scikit-learn leakage guidance and Kapoor/Narayanan research on ML reproducibility | That passing a code check alone proves unbiased evaluation |

NIST describes labeling possible outliers and investigating their causes; its modified-Z illustration uses 0.6745 * (value - median) / MAD and a potential-outlier cutoff of absolute score > 3.5. This is cited as methodological background, not adopted as a calibrated mandi cutoff.
Hyndman and Athanasopoulos discuss outliers that may be genuine events, contextual missingness and model-specific handling. Their retrospective interpolation/decomposition examples are not permission to use future data in an earlier forecast. Any such transformation used here must be restricted to the information available at that forecast origin.
The official AGMARKNET SOP's relevant price-entry wording was available in the search index; direct PDF retrieval failed in this review. Obtain and record the applicable current source schema/export semantics before claiming exact source conformity, including equality and sentinel-value handling.

## 17. How we will validate the cleaning itself
This is a planned evaluation protocol, not completed results:
1. Profile an authorized real sample across representative crops, markets, varieties, seasons, source years and coverage levels. Report gaps and source anomalies before choosing statistical thresholds.
2. Assemble an auditable review set containing both flagged and unflagged records, with deliberate coverage of genuine price spikes and seasonal changes. Compare source evidence and documented corrections. Review labels are confirmed error, supported valid observation or unresolved; independent republishers of the same source are not independent confirmation.
3. Add separate synthetic corruption tests for known duplicate events, unit swaps, date-format mistakes, missing values and invalid sensor readings. Their known truth tests implementation behavior, not the real-world error rate.
4. Measure false alarms on reviewed valid records, detected confirmed errors, unresolved-review volume, and changes in usable coverage by market/crop/season. Estimate precision/recall only on the appropriately reviewed set, disclose its sampling limits, and do not treat unresolved records as correct or incorrect by default.
5. Compare a basic-rules-only forecasting pipeline against additional cleaning/imputation candidates using the same model, data availability assumptions, chronological validation folds and observed evaluation targets. Report MAE in an explicit price unit and coverage/abstentions; do not improve scores by silently discarding difficult target dates or true spikes.
6. Tune optional statistical thresholds and imputation on training/validation periods only. Preserve a later untouched final test period; report the evaluation range and model/preprocessing versions. Cleaning of confirmed evaluation-label errors requires independent evidence and a recorded dataset revision, not disagreement with predictions.
7. Test deterministic replay, original-to-normalized traceability, row/count reconciliation and changes in downstream versions. The same input plus rule version must reproduce the same output.

Before training or adding a learned preprocessing component, document what it learns, inputs/outputs, why it is needed, cited basis, chronological evaluation plan and how it differs from the deterministic cleaning path. Dataset access and the review audit precede any claim that the cleaning procedure is validated for KisanNiti.

## Sources and existing project basis
- Repository consolidated architecture: docs/architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md, especially sections 9–10, 16–20, 28, 47–49 and 60–64.
- Current chat Layer 1: KisanNiti_Layer_1_Working_Baseline.md, version 1.2.
- Pydantic strict validation: https://pydantic.dev/docs/validation/latest/get-started/why/
- DuckDB Parquet support: https://duckdb.org/docs/stable/data/parquet/overview
- AGMARKNET data-entry SOP (relevant wording retrieved from search index; direct PDF inaccessible in this review): https://www.agmarknet.gov.in/DailyDataEntry/Docs/SOP_DEO.pdf
- NIST, Detection of Outliers: https://www.itl.nist.gov/div898/handbook/eda/section3/eda35h.htm
- Hyndman and Athanasopoulos, Forecasting: Principles and Practice, section 13.9: https://otexts.com/fpp3/practical.html
- scikit-learn, Common pitfalls and recommended practices: https://scikit-learn.org/stable/common_pitfalls.html
- Kapoor and Narayanan, Leakage and the Reproducibility Crisis in ML-based Science: https://arxiv.org/abs/2207.07048
- The anomaly and sensor-age numbers above are explicitly project development proposals, not facts sourced from these software documents or validated agricultural guidance.
