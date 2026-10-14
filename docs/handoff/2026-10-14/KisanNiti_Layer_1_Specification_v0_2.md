# KisanNiti — Layer 1 specification v0.2
Superseded for this chat's working design by KisanNiti_Layer_1_Working_Baseline.md on 13 October 2026. Retained as discussion history.
Date: 13 October 2026
Status: Reviewable reference specification. Existing architecture decisions remain authoritative. New reference assumptions are user-authorized for design; they are not CONFIRMED facts about a particular FPO.
User directions incorporated: broaden crops/regions; collect substantial useful data; research operational practice and explicitly assume a reference workflow where direct FPO facts are unavailable; make errors reproducible and maintainable.
Replaces the earlier Layer 1 review draft as the current discussion artifact, not the repository architecture baseline.

## 1. Layer boundary
Layer 1 establishes source identity, acquired facts/estimates, collection frequency, ownership, record identity, units, source-time semantics and traceability.
Basic format validation occurs at ingestion. Statistical cleaning, imputation, features, model training/selection and storage-risk classification remain subsequent-layer concerns.
Approval freshness actions and optimizer economics remain later-layer policy decisions.

## 2. Broad coverage decision
Do not hard-code Punjab, paddy, a demo FPO, a fixed mandi list, or one provider.
Maintain global/reference mandi and commodity catalogues; tenant-owned membership, lots, placements, business configurations and sales remain FPO-isolated.
Collect available authorized historical data across crops/regions in manageable source-defined export batches. Prefer an official bulk export when offered; its actual availability is not yet verified.
Production incremental jobs prioritize enabled FPO crop/market subscriptions and any separately specified forecast context markets. Broader archive expansion is a separate resumable import, not a huge request inside plan generation.
Keep source commodity, variety, grade and price units distinct. Larger raw archives do not automatically justify one blended model.
One V1 SellingPlan remains one FPO and one commodity. Broad data support does not alter that invariant.

## 3. Reference FPO operating assumptions
Classification: ENGINEERING LEAN / REFERENCE ASSUMPTION, authorized by the user to support design, not validated actual partner practice.
Documented basis: official e-NAM describes FPO aggregation and collection/sorting/grading facilities.
Reference custody: members' produce is aggregated at an FPO-managed collection/storage location before coordinated sale.
Reference ownership: farmer-linked lots remain separately accountable; do not assume purchase/title transfer to FPO.
Reference intake: a designated authorized intake operator records weighing evidence and member/lot details; manager reviews corrections and operating decisions. These role assignments are assumptions.
Reference dispatch: current recorded placement supplies origin, not the farmer address by default.
Reference transport: FPO arranges transport and records the agreed charge. Reference costing view treats that charge as an FPO planning expense; it does not decide final farmer deductions, reimbursements or settlement.
Alternative farm-held, warehouse-held and third-party locations remain representable. Each real lot must have explicitly recorded placement; no automatic collection-centre default for real records.
No evidence here establishes a universal FPO workflow, tariff, ownership-transfer rule or consent process.
Before an actual pilot, replace/confirm these assumptions against real intake, dispatch, weighing and transport records.

## 4. Organization input contract
Proposed conceptual data, not frozen database/API schema:
- FPO identity and explicit local business timezone.
- FarmerMember identity and membership within the FPO; contact details only as needed.
- ProduceLot identity, member owner, canonical commodity mapping, original source commodity/variety/grade labels when supplied.
- Registered quantity and original unit; verification source/actor/time, where supported by the reference workflow.
- Current physical placement and placement history.
- Source/import identity and who/when entered or corrected the record.
- Source evidence/attachments only where needed and access controlled.
Forms and CSV imports pass the same service validation and audit boundaries.
Available quantity is derived from authoritative lot quantity, actual sales, remaining active reservations and finalized adjustments; never an independently editable input.
Internal exact quantity unit is not finalized by this source-layer draft. Numeric precision, legal scheduling increment and model normalization are distinct later contracts.

## 5. Market source/access contract
Preferred source family: official AGMARKNET.
Candidate access channels: authorized AGMARKNET 2.0 reporting/API/export and authorized data.gov.in resource access. Maintain adapters behind common internal records.
Verified on 13 October 2026:
- Official AGMARKNET home and reference-filter endpoint returned HTTP 200.
- Filter endpoint returned 605 unique commodity labels and 4,176 unique market IDs, including 245 Punjab market reference entries.
- These are catalogue counts, NOT reporting coverage or usable historical series counts.
- Current source commodity name is Paddy(Common); source variety is a separate field. Its commodity label must not be treated as a guarantee of one homogeneous Common variety.
- Tested official monthly history requests returned HTTP 400 with code TOKEN_OR_CAPTCHA_REQUIRED and message requiring CAPTCHA key/value.
- Tested data.gov.in request without authorization returned HTTP 400, Authorization field missing.
The exact supported automated access route is an OPEN EXTERNAL DEPENDENCY. Do not bypass CAPTCHA, invent credentials, or claim historical acquisition succeeded.
Alternative investigated: CEDA offers documented AGMARKNET APIs, but its published free-use terms specify non-commercial use and attribution requirements. Do not assume it is the commercial-production default. Schema, aggregation, current coverage and access require evaluation before adoption.
Third-party wrappers have not been audited/adopted.

## 6. Market acquisition and update settings
Classification: ENGINEERING LEAN / PROPOSED STARTING CONFIGURATION. These settings are chosen for initial engineering evaluation, not source reporting guarantees or mandatory approval rules.
- Historical boundary: source's earliest available authorized observations through latest complete local business date. Obtain a manifest recording actual returned range. No fabricated minimum number of years or universal completeness assertion.
- Fetch in source-supported bounded batches; keep completed batch checkpoints, filters, pagination and response/schema version.
- Initial scheduled refresh: 06:00 and 18:00 Asia/Kolkata, suitable to the Indian source; configurable by source.
- Initial recent correction lookback: 7 local calendar days, expanded if measured corrections occur later.
- Polling a source twice daily does not make observations intraday or guarantee two new prices daily.
- A manual refresh can use the same collector with duplicate prevention; do not retrain per refresh.
- Record successful check with no new observations distinctly from failed collection.
- Honour documented provider quotas and Retry-After. Do not invent source rate limits.
- Initial transient-failure policy: maximum 3 total attempts for safe reads; bounded timeouts; delayed retry obeying provider instructions. Authentication/CAPTCHA/schema errors are surfaced rather than repeatedly retried.
Schedules and lookback must be benchmarked against actual source delays, operating timing and permitted request volume.

## 7. Market record contract
Preserve raw payload/export according to provider permission, together with:
- provider/source and source record ID when supplied;
- original/canonical market and commodity identities;
- raw variety/grade identifiers and labels where supplied;
- observed local market date;
- min/modal/max price with explicit price unit and currency;
- arrival/traded quantities, their units and original missing/status meanings when supplied;
- first-seen/received UTC time;
- publication timestamp only if truly supplied;
- batch ID, source schema version and revision/correction history.
A candidate logical observation identity includes source + market + commodity + variety + grade + date plus additional distinctions required by actual schema. Do not deduplicate on date/market alone or merge sources without a documented rule.
Source missing/unknown labels remain explicit. No invented units, prices or timestamps.
Prediction outputs later remain distinct records from market observations.
ActualSale is transaction reality and is not a replacement for public modal-price observations.

## 8. Routing and transport data
Existing DESIGN DECISION: route provider independent from transport economics.
Google Routes is an existing preferred candidate, not a finalized supplier. Standard distance/time responses do not guarantee a suitable truck route. Google's documented specialized large-vehicle routing coverage does not include India in the currently reviewed documentation.
Origin = real lot's current recorded dispatch placement. Destination = verified candidate mandi location.
Request distance/duration/status/source-time evidence when required; caching/reuse must obey supplier terms and subsequent freshness policy.
Reference transport-cost input: a manually entered agreed trip quote, with origin/destination, applicable load/vehicle constraints, currency, included/excluded charges, source and actual quoted validity if supplied.
This is a chosen reference path for development/scenario evaluation, not a claim that every FPO uses flat trip quotes.
No fabricated rupee/km rate or automatic quote expiry. An applicable usable cost basis is required before generating estimated economics.
Actual transport expense is recorded separately; a quote is not evidence of payment.
Shared vehicle/group allocation semantics remain a later transport/optimizer contract; never double-charge a shared quote across assignments or silently invent sharing.

## 9. Sensor ingestion
Existing DESIGN DECISION: ESP32 + DHT22 -> MQTT -> SensorReading.
Chosen engineering starting setting: sample every 5 minutes, configurable. This is a development/deployment candidate that must be tested, not a scientifically justified urgency interval for every crop.
Capture device ID, event identity/sequence where supported, measured time where trustworthy, receipt time, temperature in degrees C, relative humidity percent, explicit reading/error state and assignment history.
One sensor per monitored zone initially; expanding a sensor's zone coverage does not establish that every lot shares identical conditions.
Credentials are per device/tenant assignment; exact MQTT broker/topic/auth/QoS contract must be finalized in the hardware implementation task. No credentials in published logs/artifacts.
Replay/duplicate readings must not become additional independent measurements.
Offline buffering/backfill must preserve actual measurement times rather than portray old readings as current.
Simulated/test readings are marked and isolated from real operational data.
Ambient readings do not measure direct grain moisture, spoilage or quality.
Unsupported/unvalidated crop-specific assessment remains UNKNOWN rather than assigned a fabricated threshold. Storage science is a later-layer evidence gate.

## 10. Data volume accounting
Exactly verified now: 605 commodity reference records, 4,176 market reference records and 245 Punjab market reference records.
Actual historical price-row count: UNVERIFIED because authorized report access is not established.
No estimate may be presented as downloaded records.
Every import manifest must report:
- attempted and successful batches;
- raw records received, unique records and corrections;
- rejected/quarantined records with reasons;
- observation date range;
- reporting markets/commodities/varieties/grades;
- per-series observed count and missing calendar positions;
- raw and processed bytes;
- latest observation/collection times and source delays where measurable.
Capacity example only: 100 markets, 3 distinct market/crop/variety/grade series in each, and 1 Jan 2021–31 Dec 2025 gives 100 * 3 * 1,826 = 547,800 calendar positions, not guaranteed observations.
5-minute sensor example: 288 scheduled readings per sensor per day; 105,120 per non-leap year with uninterrupted operation.
Route-query count depends on unique origin/destination/departure/options requests and legal reuse, not number of price observations.
Do not multiply all commodities by all markets and claim every combination exists.
No model/data sufficiency guarantee follows from a raw row total; assess seasons, freshness, target horizons and effective independent samples.

## 11. Maintainability and incident investigation
Existing foundation: modular FastAPI backend, PostgreSQL, provider/domain separation, exact inventory, auditability, explicit failures, FPO isolation.
Layer 1 implementation acceptance requirements:
- source dictionary and mapping versions;
- configuration with clear owner, reason and version;
- request/batch/record correlation IDs;
- structured error classification, e.g. authorization required, source timeout, schema changed, unit missing, mapping unresolved;
- redacted operational logs and protected raw records;
- resumable imports and safe duplicate handling;
- saved source fixtures and adapter contract tests;
- backup/restore and correction procedure;
- collection dashboard with success/failure, source date, received date, counts, rejected rows and backlog;
- runbook per provider for diagnosis, safe replay and recovery;
- explicit upgrade/migration notes for changed contracts.
No promise that any person can repair any issue. A new developer should be able to identify the failed stage, reproduce it with authorized evidence, and apply the documented repair procedure without guessing.
Do not label entire product production-ready from Layer 1 decisions or an academic simulation.

## 12. Decision/dependency classification
Resolved for the reference design:
- broad crop/region catalogue and acquisition capability;
- source/domain separation;
- FPO collection-centre aggregation reference workflow;
- designated intake staff plus manager reference roles;
- FPO-coordinated, quoted transport reference path;
- refresh and correction starting settings;
- 5-minute initial sensor setting;
- required provenance, volume manifests, replayability and error reporting.
OPEN EXTERNAL DEPENDENCIES:
- supported authorized historical/current source access;
- actual historical observation counts and coverage;
- route supplier suitability/cost/access;
- physical hardware deployment evidence.
REFERENCE ASSUMPTIONS requiring partner confirmation before real deployment:
- custody/dispatch, intake authority, transport coordination and expense ownership.
Later-layer decisions intentionally outside this document:
- statistical cleaning/imputation and training windows;
- supported forecasting models and validation gates;
- agricultural storage assessment thresholds;
- economic/fairness adoption, sharing semantics and scheduling increment;
- approval freshness actions, consent and off-plan sale reconciliation.
Do not mark these later-layer concerns as resolved by source collection.

## 13. Sources
- Official reference endpoint tested: https://api.agmarknet.gov.in/v1/daily-price-arrival/filters
- Official AGMARKNET frontend inspected: https://agmarknet.gov.in/home (public application bundle confirms report endpoint and query parameter names).
- Official OGD catalog: https://www.data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi
- Government upgrade announcement: https://www.pib.gov.in/PressReleasePage.aspx?PRID=2204750&lang=1&reg=3
- e-NAM FPO aggregation/facilities: https://enam.gov.in/stakeholders-Involved/fpos
- e-NAM operational guidelines (official indexed excerpt on weighing/assaying): https://www.enam.gov.in/web/assest/download/Revised-Operational-Guidelines-of-e-NAM.pdf
- CEDA API documentation: https://api.ceda.ashoka.edu.in/documentation/
- CEDA terms: https://ceda.ashoka.edu.in/api-terms-conditions/
- Google Routes: https://developers.google.com/maps/documentation/routes/compute_route_directions
- Google large vehicles: https://developers.google.com/maps/documentation/routes/lvr
No transport tariff, partner-specific authority or scientific sensor threshold was verified by these sources.
