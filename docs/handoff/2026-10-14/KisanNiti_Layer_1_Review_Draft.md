# KisanNiti — Layer 1 review draft
Date: 13 October 2026
Status: Discussion proposal. New recommendations are not approved production decisions.
Scope: Data sources, acquisition, update pattern, and traceability. Detailed cleaning, forecasting/model selection, storage assessment, optimization, approval, and sales behavior belong to subsequent layers.

## Authority and classifications
Existing baseline reviewed: /Users/arijitsingh/Documents/sem_7/kisaniti/docs/architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md
Also reviewed: repository AGENTS.md, blueprint status, evidence register, unknown/blocker register, and revised/rejected register.
Preserve baseline classifications: CONFIRMED, DESIGN DECISION, ENGINEERING LEAN, UNKNOWN.
New recommendations below are ENGINEERING LEAN / PROPOSED unless explicitly described as an existing DESIGN DECISION.
This draft does not supersede the architecture or resolve real-FPO unknowns.

Correction to earlier chat: the consolidated architecture specifies current V1 per-farmer true leximin fulfilment fairness (§34); real-manager acceptance remains unknown. It is inaccurate to say that the current architecture has no specified fairness formula.

## 1. Four input groups
1. Organization, farmer, produce, placement, and business reference records.
2. Mandi observations and mandi reference information.
3. Routing information plus FPO/transporter transport-pricing evidence.
4. Storage environment readings.

These are four conceptual groups, not a claim that exactly four APIs or databases supply all inputs.
Actual sales later feed inventory and evaluation; they remain distinct from market-wide observations.

## 2. Organization and produce
Existing DESIGN DECISION: FPO-scoped FarmerMember and ProduceLot. One farmer can own multiple lots. Inventory is quantity-based; available quantity is derived from authoritative quantities, actual sales, active remaining reservations, and finalized adjustments.
Existing direction: FPO staff represent farmers in V1; exact roles and authorities are UNKNOWN.
PROPOSED acquisition: staff forms; reviewed CSV import for bulk initial records. Import must not bypass the same validation/audit boundaries as forms.
Proposed information categories: FPO/member identity; lot identity/owner/commodity; entered quantity and original unit; source registration evidence; current placement where known; variety/grade when supplied, without inferred labels.
UNKNOWN: actual intake process, verification authority, required intake fields, custody, dispatch origin, quality effects, canonical quantity unit and scheduling increment.
Update pattern: when business facts change, with auditable corrections. No independently editable available quantity.

## 3. Market observations
Earlier project chats identify AGMARKNET as the market source and modal price as the first forecasting target. Existing experimental scope: Punjab / Paddy / Common variety. This is not established as the final operational pilot.
PROPOSED source family: official AGMARKNET, with its official access routes evaluated. data.gov.in is a candidate access channel to AGMARKNET data, not an independent observation source by default.
PROPOSED acquisition: initial relevant historical import plus scheduled acquisition of newly reported observations. Re-fetch a documented recent period to detect delayed/corrected reports while preserving earlier versions.
Start by evaluating daily batch collection. Exact poll times, lookback for corrections, and freshness actions remain UNKNOWN until access/reporting measurements and operational evidence.
Preserve source market and commodity IDs/names, date, variety/grade where exposed, min/modal/max prices and units, arrivals and units where exposed, and source status. Exact provider schema must be tested.
Collect arrivals if available, but do not equate arrivals with physical capacity or automatically adopt them as model features or hard optimization limits.
Government describes AGMARKNET 2.0 as providing real-time information; the OGD catalog describes daily observations. Dataset granularity, publication delay, and our collection frequency are distinct. No continuous live feed or delivery guarantee has been verified.
Official website/API access has not been proven usable in this session. Browser retrieval of the portal failed; one data.gov.in page did not expose the actual dataset. Source-family recommendation is stronger than evidence for a particular endpoint.
Historical training and production inference remain separate. Recent observations feed predictions without necessarily retraining the model.
Proposed traceability additions: observation date and first-seen/ingested time; preserve original source timezone/date semantics and publication time only when actually supplied.

## 4. Mandis and amount of history
Existing DESIGN DECISION: manager controls candidate-mandi and selected-lot boundaries for each plan.
PROPOSED operational configuration: maintain the FPO's relevant candidate-mandi list, with verified market identity/location and commodity suitability.
Audit historical coverage separately for each market/commodity/variety. Do not pick the pilot mandi list solely from the cleanest data.
Acquire relevant available history and measure seasons, gaps, changes, and recency. Exact market count and training date range remain evidence-dependent.
Model training context can include additional relevant markets without making them eligible selling destinations.
No assumption that all India, all crops, a fixed number of mandis, or a fixed number of years is required.

## 5. Routing and transport prices
Existing DESIGN DECISION: provider-independent RoutingService supplies distance/duration/status/provenance/time. TransportCostService remains separate.
Existing preferred V1 provider may be Google Routes; provider is not frozen.
PROPOSED acquisition: request origin-to-candidate-mandi estimates when required, reuse only within provider terms and subsequently agreed freshness policy.
Actual origin remains UNKNOWN until real placement/dispatch practice is established.
Transport pricing must come from validated FPO/transporter rates, quotes, or records. Collect price basis/unit, source, covered quantity/vehicle/origin/destination as applicable, and actual validity conditions; exact required fields follow real pricing.
No formula is frozen. Route distance does not establish rupee cost.
Google's current large-vehicle routing documentation lists geographic availability in the contiguous US and Japan, not India. Standard driving estimates must not be described as validated Indian truck routes.
Do not treat provider error as zero distance or physical road blockage.

## 6. Storage environment
Existing DESIGN DECISION: ESP32 + DHT22 -> MQTT -> SensorReading -> StorageConditionService -> StorageAssessment.
Layer 1 preserves environmental readings; later processing derives assessments.
DHT22 measures ambient temperature and relative humidity, not direct grain moisture, quality, or spoilage.
Existing DESIGN DECISION: placement and device assignment have history.
PROPOSED reading metadata: device identity, measurement time where trustworthy, receipt time, temperature/RH with units, reading status; deployment/assignment context and event identity where supported.
UNKNOWN: sampling rate, MQTT QoS/topics/authentication, device placement, calibration/deployment arrangements, storage thresholds, staleness rules.
Development simulations must be visibly separated from actual measurements and must not enter real operational decisions as live data.
Lack of usable evidence must remain explicit; no low-urgency default.

## 7. Common collection behavior
Existing DESIGN DECISION: bounded timeouts/retries where safe, validation/normalization, source/status/time, canonical IDs, explicit failures, FPO isolation, UTC technical timestamps and explicit local business dates.
PROPOSED: retain original responses/imports alongside mapped records when licensing and retention rules permit; record ingestion batches and preserve source corrections.
Do not invent publication timestamps, arrival units, zero fallbacks, or unexplained history merges.
Detailed transformations and model-specific datasets are Layer 2 decisions. Basic format checks necessarily occur during ingestion.

## 8. Reuse evaluation
Prefer maintained existing implementations where they fit, after reviewing provenance/license, relevant coverage, reproducibility, failure behavior, security, and integration burden.
A paper/model architecture, training code, pretrained weights, and an actively maintained production service are different things.
Candidate research found: Ma et al., An Interpretable Produce Price Forecasting System for Small and Marginal Farmers in India using Collaborative Filtering and Adaptive Nearest Neighbors (2018/ICTD 2019). It uses AGMARKNET, sparse-data treatment and interpretable forecasts; it is a candidate for later methodology evaluation, not an adopted model or verified ready-to-use current pretrained artifact.
Third-party AGMARKNET wrappers were discovered but not audited or adopted. Do not equate a wrapper's claim of verified endpoints with an official supported integration guarantee.

## 9. Questions to resolve with the user/FPO next
1. Is there an actual pilot FPO, or is the present phase academic design without an operational partner?
2. Where is participating produce physically stored before dispatch, and can origins vary?
3. Continue Punjab / Paddy / Common as the first data audit, or use another commodity/region?
4. Who verifies quantity and how is transport arranged/priced? Obtain real records before freezing production behavior.

## Sources and what was verified
- Official OGD catalog: https://www.data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi — indexed description identifies AGMARKNET-generated daily min/modal/max wholesale prices; direct retrieval was unsuccessful.
- Government AGMARKNET 2.0 announcement: https://www.pib.gov.in/PressReleasePage.aspx?PRID=2204750&lang=1&reg=3 — successfully opened; documents November 2025 upgrade and on-the-spot entry and real-time information claims, not API delivery guarantees.
- Official AGMARKNET SOP: https://www.agmarknet.gov.in/DailyDataEntry/Docs/SOP_DEO.pdf — search-index excerpt describes editing/freezing rules and no-transaction reporting; direct retrieval unsuccessful.
- Google route documentation: https://developers.google.com/maps/documentation/routes/compute_route_directions — successfully opened; verifies route distance/duration.
- Google large vehicle routing: https://developers.google.com/maps/documentation/routes/lvr — official indexed documentation lists limited customer access and geographic limitations.
- Research paper: https://arxiv.org/abs/1812.05173 — successfully opened abstract; no reproduction or pretrained artifact validation performed.

