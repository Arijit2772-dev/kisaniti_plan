# KisanNiti — Final Layer 1 Design v1.2

> **14 October 2026 integration update:** Start with [Integrated Layer 1–3 handoff](KisanNiti_Layers_1_to_3_Handoff.md) and its acceptance plan. They consolidate this document and explicitly supersede obsolete hardware/location assumptions. Current hardware: ESP32-WROOM-32 + protected I2C SHT30 probe, with no manual moisture-meter requirement. Older hardware proposals below are retained only as history. No implementation/test completion is implied.

Date: 14 October 2026 (integration clarification; original acquisition window remains the 13 October baseline, with rolling updates thereafter)
Status: Final selected design baseline for Layer 1: data sources, collection, storage and update handoffs. Implementation and source acquisition remain to be carried out.
Authority: Incorporates the user's explicit direction to choose a concrete approach, broaden data, and use labelled dummy organization data and reasonable documented assumptions.
This is not a claim that data has been downloaded, code implemented, models evaluated, or a real FPO workflow validated.
Supersedes Layer 1 discussion drafts in this chat. The repository architecture remains the baseline for all unaffected concerns.

## Purpose and boundary
Cross-layer acceptance requirement added 14 October 2026: completion requires component, adjacent-layer integration and end-to-end evidence under [Integration, Testing and Hardware Guide](KisanNiti_Integration_Testing_and_Hardware_Guide.md). Simulator success and real-source/device success are reported separately.
Layer 1 supplies traceable facts for the other four layers: validation, optimization, manager decisions, and communication/actual-sales feedback.
Its deliverable is a reproducible data foundation, not a trained forecasting model or a live deployment.
Here, "first phase" means this data layer; it does not rename the repository's separately governed implementation phases or authorize product code before its blueprint gate.

The fixed design below separates three kinds of statements:
- Selected settings: quantities, schedules, formats and adapters chosen for this project.
- Verified facts: source responses and documentation actually inspected.
- Acceptance evidence: access, real coverage, performance and business suitability that implementation must demonstrate.

Broad national collection does not mean every crop has a usable forecast. Store all eligible available observations; expose forecasting/planning support only for combinations that pass the later data-quality and model-evaluation gates.

## 1. Fixed source strategy
Layer 1 has four groups:
1. FPO/member/produce/placement facts.
2. Market observations.
3. Route estimates and transport pricing inputs.
4. Environmental sensor readings.
Synthetic development records and real observations must have explicit origin labels and separate datasets. A real observation used in a demo remains a real observation; attaching a synthetic lot does not make it a real deployment.

## 2. Organization data: generated fixtures first
Development source: a reproducible seeded generator, not a prerequisite spreadsheet or partner organization.
Normal development profile:
- 5 fictional FPOs.
- 200 fictional farmer members per FPO = 1,000 members total.
- 3 initial lots per member = 3,000 lots total.
- 2 fictional collection centres per FPO = 10 centres.
- 2 monitored zones per centre = 20 zones.
- 1 simulated device per zone = 20 devices.
- 10 commodity categories for demo breadth, mapped explicitly to source commodity IDs: paddy, wheat, maize, onion, potato, tomato, cotton, soyabean, arhar and gram. Exact source names must be resolved in the mapping, not guessed.
- Each FPO has its own selected candidate-mandi list of 20 entries for the fixture profile; this is a fixture size, not a production eligibility rule or platform cap.
Use clearly fictional names and non-deliverable contact fields; external communication is disabled for fixture identities.

A separate generated stress profile scales to 50 FPOs / 10,000 members / 30,000 initial lots. It is a test workload, not customer or farmer adoption.
Do not generate millions of farmer rows just to inflate a training-data claim.

Reference operations: FPO-managed collection centre, designated intake operator, manager governance, FPO-coordinated transport. Preserve explicit placement per lot and alternative location capability. Farmer ownership of lots remains traceable; no assumed transfer of title.

Later real source: authenticated application forms. CSV/spreadsheet import is an optional bulk-input convenience and uses the same validation/audit boundary.
Minimum operational entities: FPO, farmer member, produce lot, storage location/zone, device assignment, transporter quote and actual sale. Lots record commodity/variety/grade when known, quantity with unit, farmer/FPO linkage, explicit location, relevant dates and evidence/source. Unknown quality or dates stay unknown.
Each actual-sale entry records its lot, sold quantity and unit, sale date, realized price/amount and currency where supplied, destination and source evidence where available. A sale is an operational outcome, not a substitute for an official mandi observation.
Inventory remains exact and transactionally derived. Preserve the existing architecture's separation between original quantity, sales, reservations and adjustments.

## 3. Market data: maximum useful real coverage
Chosen primary source: official AGMARKNET observations through supported authorized reporting/export/API access.
Chosen supplementary research channel: CEDA Agri Market Data, when available at required mandi/day detail and used within its published non-commercial/attribution terms. Its state/district/monthly aggregates must not be substituted for mandi/day observations.
Production commercial use must have a suitable source/access basis. CEDA is not automatically adopted as a commercial production dependency.

Acquisition target:
- Geography: all available Indian states/UTs and reporting mandis.
- Commodity scope: all available commodity records; no Punjab/paddy-only hard-code.
- Initial archive search window: 1 January 2000 through 12 October 2026 inclusive.
- Then continue to the latest completed India business date as time advances.
- Preserve separately reported variety, grade and units.
- Keep all legitimately reported observations, subject to source permission. Do not manufacture missing years or records.
- Priority order for a manageable build: the most recent ten years, followed by older available history. The priority affects collection order, not an assertion that older data always improves a model.
- Batch by source-supported geography/commodity/time partitions; save checkpoints. Prefer bulk exports where the provider offers them. Do not assume a bulk API exists without verification.

The archive size target is all retrievable eligible records in that scope, not a guessed row number. Millions of records are a project objective; exact imported, deduplicated and usable totals must come from the manifest.
No universal minimum record count guarantees an accurate forecasting model.
Do not multiply every market by every commodity and every day and call that real data.
Historical archive scope is not automatically the final training window. Each selected model must be evaluated using appropriate series and time-based test periods.

Market fields:
- source/provider and provider record ID if supplied;
- original/canonical state, district, market and commodity identities;
- source variety and grade values, including explicit missingness;
- market observation date;
- minimum, maximum and modal price;
- price currency and denominator unit;
- arrivals/traded quantities and units only where genuinely exposed;
- first-received time, source publication time only when supplied, import batch ID;
- original record and revision/correction history where allowed.

The records are daily market summaries, not individual sale transactions. ActualSale remains separate.

## 4. Current market updates
Chosen starting collector configuration:
- Scheduled checks at 06:00 and 18:00 Asia/Kolkata.
- Re-fetch the preceding 7 days to detect late records and corrections.
- Additional manual refresh uses the same safe collector.
- Source date and collection date are shown separately.
- Prioritize enabled FPO market/commodity subscriptions and needed forecasting context; national archive expansion runs separately.
- Source-supported batch limits and quotas govern request volume.
- Safe transient read failures: at most 3 total attempts, respecting Retry-After; authorization/CAPTCHA/schema errors require action rather than repeated attempts.
These are engineering operating settings, not a statement about publication guarantees or approval freshness rules.

## 5. Verified source access status
Tests on 13 October 2026:
- Official reference endpoint returned 605 unique commodity records and 4,176 unique market IDs, including 245 Punjab market entries.
- Reference catalogues do not establish historical coverage.
- Commodity Paddy(Common) is distinct from its separately reported variety.
- Official monthly history endpoint rejected unauthenticated report access with TOKEN_OR_CAPTCHA_REQUIRED.
- The tested data.gov.in request without authorization returned Authorization field missing.
Historical ingestion is not complete. Supported authorization/export access is an external dependency, not something a design decision can fabricate.
No source protection is to be bypassed.
A source connector and access test must prove exact schema, units, pagination, allowed use, corrections and latest coverage before automated use.

## 6. Routing and transport
Chosen first real route adapter: Google Routes, behind the already-decided provider-independent RoutingService.
Before live operation, verify account access, price/budget, source terms and suitability for the actual vehicle/context.
Standard driving distance/time is a planning estimate. It is not certified Indian truck navigation; currently reviewed specialized Google large-vehicle coverage does not include India.

Development:
- Deterministic route fixtures, explicitly marked synthetic.
- 10 fixture origins x 20 associated candidate mandis = 200 origin-destination route fixtures.
- 200 corresponding synthetic quote fixtures for integration scenarios.
- Fixture source/status/unit/quantity applicability must be explicit.
- Failure scenarios and missing-route cases are separate fixtures, not disguised as successful 0 km routes.
Real routing requests happen when planning requires them; reuse only within supplier terms and the later agreed freshness policy.

Cost input:
- Start with recorded transporter quotes, not a purported universal per-km rate.
- Development quotes are synthetic test values; real quotes are entered by authorized staff.
- Record origin/destination, applicability to load/vehicle, currency, included/excluded charges, source and validity as actually supplied.
- Actual cost is distinct from an estimate.
- Shared-trip costing belongs to the later transport/optimizer contract. Never double-charge a shared quote or assert a fixed truck quote applies to any quantity.
FPO coordination/payment is a reference assumption. Actual settlement among farmers is not inferred.

## 7. Sensor source and exact fixture volume
Development source: seeded environmental-reading simulator using the same conceptual reading contract as the real hardware.
Fixed normal fixture profile:
- 20 devices.
- Every 5 minutes.
- 30 days.
- 20 x 288 x 30 = 172,800 baseline scheduled readings.
Generate a separate fault corpus for duplicates, invalid readings, missing periods, delayed uploads and device reassignment. Those faults must not silently alter the stated baseline count.
The timing is a selected engineering starting point, not a scientific storage-risk interval.

Real source: ESP32 + DHT22 via MQTT.
Hardware recommendation under review, 14 October 2026: because no hardware has been purchased and ease of assembly/maintenance is now the priority, the [Integration, Testing and Hardware Guide v0.2](KisanNiti_Integration_Testing_and_Hardware_Guide.md) recommends ESP32 Feather V2 + connector-equipped SHT40 and ESPHome for initial connected bring-up. This explicitly revisits the earlier DHT22 purchase recommendation; the normalized temperature/humidity contract and real/synthetic separation remain required. Offline persistent replay remains an unimplemented reliability requirement, not an assumed ESPHome capability.
- Ambient temperature in degrees C and relative humidity in percent.
- Device identity, measurement time where trustworthy, receipt time, event identity/sequence where supported, assignment history and reading status.
- Start with one device per monitored zone.
- Offline replay preserves original measurement times.
- Keep simulator origin explicit and isolated from real operational decision evidence.
- Sensor readings do not directly measure grain moisture, quality or spoilage.
Crop-specific assessment requires subsequent evidence; unsupported assessments remain UNKNOWN.

## 8. Storage, volume and recovery
Use the repository's modular FastAPI/PostgreSQL architecture for operational/reference state.
For the historical archive, select compressed partitioned Parquet for processed research datasets and retain original permitted exports/responses separately. Partition sizes should be measured; do not make one file per price observation.
This is an archive-format choice, not a reason to add distributed infrastructure.
Store large research data outside source-code Git; commit dictionaries, mappings, manifests and generation scripts/fixtures where appropriate.
No GB/TB requirement is claimed before measuring actual data. Record compressed/raw bytes and required workspace budget during acquisition.

Every dataset release has:
- source and permitted-use notes;
- exact request/import scope;
- original and clean record counts;
- duplicate/correction/rejection counts;
- actual date range and coverage by series;
- schema/mapping/transformation versions;
- file checksums and creation time;
- real/synthetic/mixed-context origin labels;
- reproducible reconstruction instructions.

Every import has batch/request IDs, resumable checkpoints, explicit errors and a repair runbook.
A rerun must not silently double-count data.
Credentials and personal data must not appear in public logs.

## 9. Existing pretrained models: selected evaluation strategy
Select Amazon Chronos-2 as the first pretrained time-series benchmark because published weights, code and an Apache-2.0 model card are available.
It is a general forecasting model, not verified here as a pretrained mandi-specialist.
No claim that it is best for KisanNiti is established.
Later forecasting work compares:
- persistence/moving-average baselines;
- a trainable tabular model such as XGBoost using real mandi lag features;
- Chronos-2 with real historical context;
- A3T-GCN, the architecture's explicit graph-model candidate, if justified by data and research objectives.
Model promotion requires held-out time-based results, cost/latency and applicable coverage.
Check pretrained data provenance/cutoffs when designing evaluation; do not ignore possible overlap with historical tests.
Only describe our own actual operations: trained, fine-tuned, evaluated or used for inference.
Do not claim upstream pretraining corpus size as KisanNiti's own training dataset.
Dummy FPO/route/sensor data is for workflow/integration/load tests, not evidence of price-forecast accuracy.

## 10. Collection contract and handoff to validation
Every connector delivers a consistent envelope: source, real/synthetic origin, source-record identity when available, FPO scope when applicable, observation/measurement time, receipt time, units, schema version and batch/request identity. Preserve original source values and missing fields explicitly.
Store technical timestamps in UTC; interpret market business dates using their documented local date conventions, with India business scheduling in Asia/Kolkata.
Mappings from provider labels to canonical IDs are versioned. Ambiguous names are quarantined for resolution; similarly spelled mandis or crops are not automatically merged.
Retain permitted originals separately from normalized records. Convert units only through explicit documented conversions. Never silently treat missing prices as zero, forward-fill them as observed prices, or overwrite a corrected observation without revision history.
Cross-source duplicates need provenance and reconciliation; do not sum overlapping AGMARKNET and CEDA rows as independent observations.

The ingestion boundary checks readability, required identifiers, schema and units, records duplicates and revisions, and quarantines malformed records. Layer 2 defines the detailed crop/market validity, outlier, missingness, freshness and model-readiness policies. Suspicious records remain auditable rather than disappearing.
Historical evaluation must distinguish the date an observation describes from the date it became available. If historical publication times or revisions are unavailable, document that limitation; today's corrected archive is not proof of exactly what a past forecast could have known.

## 11. What happens when data changes
Training, prediction and optimization are separate operations:
- Training/fine-tuning learns or updates a model from historical data. It is a separate evaluated job, never an automatic action for each record, user request or replan.
- Prediction runs an already selected model with relevant recent inputs. A general pretrained model can be evaluated without first training it from scratch.
- Optimization uses usable predictions plus inventory, transport inputs and supported storage assessments to propose an allocation on a manager's planning/replanning request.

| New input | Immediate work | Downstream work |
| --- | --- | --- |
| Changed market observations | Validate, store and version the changed batch | Queue predictions only for supported series whose model inputs changed, including declared cross-market dependencies |
| New or corrected lot | Validate and update authorized operational records | Update available inventory; make the change visible to a future plan |
| Actual sale, reservation or adjustment | Apply the authoritative inventory transaction | Refresh availability; retain actual outcomes for later evaluation |
| Sensor batch | Validate, deduplicate and associate readings with the correct device/zone assignment | Refresh supported storage assessments; unsupported assessments remain UNKNOWN |
| Route result or transporter quote | Store permitted result/quote with status and applicability | Refresh relevant transport estimates for a future plan |

Do not forecast once per imported row. Collect a batch, identify changed model inputs and combine repeated pending requests for the same forecast input version. Skip duplicate or unchanged records. Complete a relevant historical import partition before preparing dependent model inputs.
Archive expansion is a separate background workload from active FPO updates. Active forecasting uses the selected model's required context, not the entire national archive on every update.
Use recoverable background jobs with recorded status, attempts, input versions and errors. A safe retry must not duplicate records, sales or inventory effects. Implementation should fit the modular monolith; large data volume alone does not require a new distributed platform.

The application exposes source observation date, last successful collection, forecast input cutoff/generation time/model version, and pending/failed/unavailable status. Previously completed results must retain their timestamps; a failed refresh does not make an old result fresh.
A forecast is stored as a derived forecast, never written back as an observed mandi price.
Before planning, apply the later agreed usability/freshness rules. Record the inputs used for each proposed plan. Approval must recheck current inventory transactionally.
New data cannot silently rewrite an approved plan. Approval/reservation and replanning follow the existing architecture; subsequent decisions create the appropriate new revision/cycle.

Starting runtime schedule: market collection twice daily; simulated/real sensor sampling every five minutes; operational edits and actual sales processed when submitted; route requests when required for planning. Price prediction follows relevant successful batches; optimization follows an explicit plan/replan request.
The forecasting stage will choose any training/fine-tuning cadence and promotion rules after model evaluation. Layer 1 fixes the trigger separation rather than inventing a model lifecycle before selecting a model.
Measure collection, prediction and optimization latency separately on representative workloads. No fixed response-time, hardware-cost or model-accuracy claim is established by this specification.

## 12. Acceptance criteria and implementation order
Layer 1 implementation is ready to hand off when the following evidence exists:
1. Versioned data dictionary, source-to-canonical mappings and explicit unit/date conventions.
2. Reproducible normal, stress and fault fixtures with the specified counts, origin labels and disabled external communication for fictional identities.
3. Authorized market-source access, documented terms/quotas, and a successful bounded import proving schema, units, date coverage and resumable pagination/export handling.
4. A real historical dataset manifest reporting requested versus acquired coverage, actual bytes, original/normalized/usable counts and every missing/rejected partition. It must make no unverified claim of complete nationwide history.
5. Incremental collection demonstrated with new, unchanged, corrected, late and failed records; replay without duplicate effects and recovery from interruption.
6. Routing and sensor adapters exercised with their fixtures and failure scenarios. Before enabling their real mode, verify provider credentials/budget/terms and hardware identity, timestamps and assignments respectively.
7. Evidence that organization data stays isolated by FPO and real/synthetic data cannot be silently confused; access checks, logs and exports protect credentials and personal data.
8. A durable downstream change handoff demonstrated without per-record model retraining or automatic mutation of approved plans.
9. A repair guide covering access expiry, source/schema changes, missing records, unit/mapping problems, delayed sensors, rejected batches, replay and dataset rebuilding, plus a tested restoration path for operational data.

Build order: contracts and fixtures; bounded real source access/import; recent-history expansion and manifests; older available archive; incremental updates and recovery; downstream handoff checks. A failed access prerequisite is reported explicitly while independent fixture/contract work proceeds.
Design finalization is complete with this document. Dataset acquisition, fixture generation and these acceptance checks are not claimed complete.

## 13. What this baseline settles
- Development organization source: generated fixtures.
- Real organization source: application entry, optional imports.
- Exact normal/stress fixture sizes.
- Broad national archive acquisition scope and fixed initial date window.
- Chosen market source family/access-channel policy and explicit authentication dependency.
- Fixed initial market and sensor collection schedules.
- First routing provider and explicit development mocks.
- Quote-based transport input with synthetic fixtures for development.
- Sensor simulation volume and future hardware path.
- Provenance, accounting, replay and storage format requirements.
- Minimum collection contracts, safe update triggers and downstream responsibilities.
- The evidence and recovery documentation needed to accept Layer 1 implementation.
- Pretrained-model evaluation inclusion, without inventing a winner.

Actual downloaded counts, permitted authorized access, real hardware deployment and forecasting results are measurements/dependencies, not numbers we can decide into existence.
This design does not claim all crops have trained models, all mandis are eligible for an FPO, or the product has passed production validation.

## 14. Integration commitment and component status
The intended deliverable is one connected workflow: collect facts; validate them; derive usable forecasts/storage assessments/transport estimates; optimize; obtain a manager decision; communicate the approved instructions; record actual sales and reconcile stock for the next cycle. All required business stages must participate in an end-to-end acceptance scenario. Optional model experiments do not become mandatory production dependencies just because they were named.

The repository architecture states: "A3T-GCN is a candidate, not a domain dependency." This clarification preserves that decision. None of this document's selected or candidate components is being reported as implemented based on its inclusion here.

| Component | Job | Status |
| --- | --- | --- |
| Collection adapters and validation boundary | Convert source-specific data into traceable, usable records | Required application capability |
| Forecasting service | Produce normalized price forecasts | Required capability for forecast-based planning; winning model not selected |
| Persistence/moving average, XGBoost, Chronos-2 | Provide comparisons for forecast accuracy and operational cost | Evaluation candidates; not three compulsory sequential runtime steps |
| A3T-GCN / GNN | Test whether information across related markets improves forecasts | Conditional research candidate; not selected or implemented |
| Routing and transport-cost services | Supply distance/time and applicable cost estimates | Required for choices that depend on transport; provider separated from domain |
| Sensor ingestion and storage assessment | Supply environmental evidence and supported assessments | Selected capability; physical hardware can be simulated in development and unsupported assessment remains UNKNOWN |
| Optimizer | Allocate available lots across admissible market/date choices under the agreed constraints | Required; independent of which forecasting model produced its inputs |
| Manager approval, communication and actual-sale recording | Turn proposals into governed operational instructions and record reality | Required end-to-end workflow; detailed contracts belong to their later layers |

GNN means graph neural network. A possible mandi experiment represents mandis as nodes, declared relationships as edges, and historical prices/arrivals as time-varying inputs. Geographic proximity or historical price similarity are hypotheses for those relationships, not evidence of actual trade or causation. Any learned relationships must use training-period information only. The graph definition, missing-data handling and sufficient aligned market coverage must be specified before the experiment.
A3T-GCN was introduced for traffic forecasting; adapting it to mandi prices needs training and evaluation. Library implementations exist, which establishes a practical implementation path, not proven mandi accuracy or an already-trained mandi model.
Only promote it if held-out evaluation and operational measurements justify the added complexity. If it fails that test, document the result and use the better-supported model. The product still retains forecasting and optimization.

Integration rules:
1. Shared canonical FPO/member/lot/market/commodity/location IDs, explicit units and consistent dates across modules.
2. Every model adapter returns the same versioned PriceForecast contract: market, commodity, target date, predicted price with explicit currency/denominator, generation time, input cutoff/version, model version and availability/status. Variety/grade applicability must be explicit or constrained by the supported-series mapping; never apply an incompatible variety forecast silently. These are required contract semantics; exact API/schema fields remain to be frozen in the implementation blueprint.
3. Sensor, route, transport and inventory modules provide their own typed outputs with provenance/status. Raw provider payloads never enter the optimizer directly.
4. A planning service assembles validated input. The optimizer performs computation without network calls or database writes; the application handles persistence and approval transactions.
5. Manager approval rechecks/reserves available quantities. Actual-sale recording changes authoritative inventory through the defined transaction path, and subsequent plans use the remaining availability. Exact partial/off-plan sale reconciliation must be settled in the relevant later layer before implementation.
6. A mock adapter must satisfy the same contract as its real counterpart and remain labelled synthetic. Simulation cannot establish real-provider or hardware readiness.
7. Maintain versioned example requests/results and contract tests so work by multiple agents can be integrated against the same agreed boundary. Record contract changes centrally rather than changing one component alone.

End-to-end implementation evidence must cover at least: an authorized FPO lot; source observations and their validation; a usable forecast and applicable transport inputs; an optimizer proposal; manager approval and reservation; a recorded communication outcome; a partial sale; correct remaining stock; and a subsequent planning cycle. Include duplicate events, unavailable inputs and failed communication/retry cases without false success or duplicate inventory effects. Communication tests use a test destination; real sending requires an authorized configured channel.

Feasibility assessment: the architecture has a practical software implementation path with available databases, model libraries, routing services and sensor protocols. Integration success, forecasting usefulness, data access and production readiness still require the stated implementation evidence. No product code or end-to-end test was executed as part of this design clarification.

## Sources
- Official AGMARKNET reference endpoint: https://api.agmarknet.gov.in/v1/daily-price-arrival/filters
- Official AGMARKNET: https://agmarknet.gov.in/home
- Official OGD catalog: https://www.data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi
- CEDA coverage description: https://ceda.ashoka.edu.in/data-portal/ (describes 300+ commodities, 2,700+ mandis and long historical coverage; page references 2000 in overview and 2001 in a detailed section, so exact per-series start dates require inspection)
- CEDA API: https://api.ceda.ashoka.edu.in/documentation/
- CEDA published terms: https://ceda.ashoka.edu.in/api-terms-conditions/
- e-NAM FPO workflow basis: https://enam.gov.in/stakeholders-Involved/fpos
- Google Routes: https://developers.google.com/maps/documentation/routes/compute_route_directions
- Google large vehicle routing: https://developers.google.com/maps/documentation/routes/lvr
- Chronos-2 model card: https://huggingface.co/amazon/chronos-2
- Chronos implementation: https://github.com/amazon-science/chronos-forecasting
- GNN implementation documentation: https://pytorch-geometric.readthedocs.io/en/latest/get_started/introduction.html
- A3T-GCN research paper (traffic forecasting): https://arxiv.org/abs/2006.11583
- A3T-GCN implementation documentation: https://pytorch-geometric-temporal.readthedocs.io/en/stable/modules/root.html
