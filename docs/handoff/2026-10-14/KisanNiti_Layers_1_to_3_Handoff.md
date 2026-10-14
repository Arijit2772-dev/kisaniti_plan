# KisanNiti — integrated Layer 1–3 handoff

Revision 1 • 14 October 2026 • Design review and implementation handoff; not an implementation completion report.

## 1. Read this first

KisanNiti helps an FPO manager decide **which selected farmer lots to send to which candidate mandis, on which dates, and in what quantities**, using predicted prices, available stock, transport economics and supported storage assessments. The manager remains the decision-maker.

The connected design is:

```text
LAYER 1: collect facts
  Organization/stock/placement | Mandi observations | Routes + quotes | Sensors
                                     ↓
LAYER 2: preserve originals → clean → validate → assess usability
                                     ↓
LAYER 3: forecast prices + assess storage + prepare transport economics
         → freeze planning inputs → optimize → independently validate
                                     ↓
         explained, immutable proposal; stock is NOT reserved yet
                                     ↓
LAYER 4: manager decision + current checks + atomic reservation
                                     ↓
LAYER 5: communication + actual sales + inventory reconciliation
                                     ↓
                       next planning cycle
```

Layers are logical responsibilities, not five microservices. Keep the existing modular Next.js / FastAPI / PostgreSQL architecture. Offline training, scheduled imports and background workers belong to this application design; no Kafka/Kubernetes/Redis requirement has been established.

**Current completion:** documents and a circuit drawing exist. Accepted application implementation, real data acquisition, model evaluation, integrated software tests and real-device tests are not complete. The companion [acceptance plan](KisanNiti_Layers_1_to_3_Acceptance.md) states how completion must be demonstrated.

### Authority and classifications

Use four labels: **CONFIRMED** (evidence/user statement), **DESIGN DECISION** (chosen behavior), **ENGINEERING LEAN** (preferred but unproven), **UNKNOWN** (requires a decision or evidence). A selected design is not evidence that implementation works.

This document consolidates the current chat's Layer 1–3 design and explicitly supersedes its obsolete hardware and assumed universal collection-centre workflow. It does not silently rewrite the protected repository architecture or mark its blueprint frozen. The Layer 2/3 proposals remain proposals where identified below. Newly specified contract details are an **ENGINEERING LEAN for the contract-freeze task**, not a claim they were previously agreed.

Repository location: `/Users/arijitsingh/Documents/sem_7/kisaniti`.

Before repository work, read `AGENTS.md`, the complete `docs/architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md`, `docs/blueprint/00_BLUEPRINT_STATUS.md`, and relevant decision/unknown registers. The status currently says product coding is prohibited and API/database contracts are not frozen. Logical **Layer 3** here does not mean repository implementation **Phase 3** has been completed.

User-directed hardware/location changes must be recorded as explicit amendments when this handoff is promoted into the repository. Keep old decisions in history. The pre-existing untracked constraint catalogue `docs/blueprint/07_ARCHITECTURE_IMPLEMENTATION_CONSTRAINTS.md` was not modified by this review.

## 2. Decisions carried forward and corrections

| ID | Classification / status | Decision or fact |
|---|---|---|
| D01 | DESIGN DECISION | Multi-FPO system; enforce tenant access on the server. Global mandi reference data does not make private lot data global. |
| D02 | DESIGN DECISION | One plan covers one FPO, one commodity, selected lots, candidate mandis and finite dates. |
| D03 | DESIGN DECISION | Separate observations, forecasts, proposals, approvals and actual sales. |
| D04 | DESIGN DECISION | Initial organization data is explicitly synthetic; broad national market history is the real research target. |
| D05 | DESIGN DECISION | Cleaning is Layer 2; retain raw data, conversions, issues and lineage. |
| D06 | DESIGN DECISION | Training, prediction and optimization are separate jobs; never retrain per sensor reading or user request. |
| D07 | DESIGN DECISION | Hardware and simulator feed the same versioned sensor contract; source origin remains visible. |
| D08 | CONFIRMED user context | Target physical demonstration: harvested paddy/wheat in bags at farm sheds; electricity and mobile coverage available. This is not an independently surveyed FPO workflow. |
| D09 | DESIGN DECISION, latest hardware | ESP32-WROOM-32 development board + protected I2C SHT30 probe. No routine manual grain-moisture measurement. |
| D10 | DESIGN DECISION | Sensor reports temperature and relative humidity. Derived storage risk is an approximation with policy provenance; no direct spoilage, moisture-percent or shelf-life claim. |
| D11 | DESIGN DECISION | Exact stock/money, explicit remainder, traceable placement and device-assignment history. |
| D12 | DESIGN DECISION | Routes supply distance/time; applicable quotes supply transport economics. Neither missing value becomes zero. |
| D13 | DESIGN DECISION | Pure optimizer; independent deterministic input/output validators; no external calls inside them. |
| D14 | DESIGN DECISION | Allocation → per-farmer true leximin fairness → supported storage urgency → expected net selling return → deterministic tie-break, within technical/economic admissibility. Real-FPO policy acceptance remains UNKNOWN. |
| D15 | DESIGN DECISION | Proposal does not reserve stock; approval rechecks and reserves atomically; actual sales remain separate. |
| D16 | ENGINEERING LEAN | Next seven calendar dates for demo, only explicitly allowed operating dates; partial mode default; no same-day dispatch assumption. |
| D17 | ENGINEERING LEAN | Baselines, XGBoost, Chronos-2 comparison; GNN/A3T-GCN conditional; no model winner established. |
| D18 | ENGINEERING LEAN | MILP first prototype, [SCIP](https://www.scipopt.org/)/PySCIPOpt candidate; no production solver selected. |
| D19 | DESIGN DECISION, demonstration boundary | No empirical arrivals-as-capacity restriction in the starting demonstration; MarketLoadPolicy DISABLED. Mathematical tests may exercise explicitly synthetic STRICT limits. |
| D20 | DESIGN DECISION | Completion requires component, contract, adjacent-layer, full-workflow and relevant real-source/device evidence; no success inferred from a drawing. |

**Superseded:** DHT22 / DS18B20 / manual moisture-meter kit, Feather V2 + SHT40 proposal, and “all produce starts in an FPO collection centre.” None is the current purchase or universal workflow requirement. No GNN, pretrained model or solver was selected simply because it was previously named.

## 3. Layer 1 — exactly what enters the system

### 3.1 Sources, amount and timing

| Source group | Development / real source | Amount selected | Update behavior |
|---|---|---|---|
| Organization | Seeded fictional FPO/member/lot/location records; later authenticated forms and optional CSV import | 5 FPOs × 200 members = 1,000 members; 3 lots/member = 3,000 lots | On authorized submission; not a continuously polled external feed |
| Market | Authorized AGMARKNET reports/exports/API; CEDA supplementary research subject to access/terms | All retrievable permitted Indian mandi/commodity/variety/grade daily records in the target window; exact total measured after import | Start 06:00 and 18:00 Asia/Kolkata; revisit preceding 7 days for corrections |
| Route + transport | Deterministic synthetic route/quote fixtures; real provider-neutral Google Routes adapter plus recorded transporter quotes | Legacy load fixture: 10 origins × 20 corresponding mandis = 200 route records and 200 quote fixtures | Route when needed for planning; quote when received/updated; validity explicit |
| Sensors | Simulator now; ESP32 + SHT30 later | Load fixture: 20 devices × 288 events/day × 30 days = 172,800 scheduled reading events; one physical node initially | Five-minute starting sample interval; event-driven ingestion |

“Reading event” above means one scheduled temperature/humidity sample pair. It does not mean 172,800 independently verified crop outcomes. Fault events are generated separately and counted separately.

Stress organization fixture: **50 FPOs, 10,000 members, 30,000 lots**. These are workload sizes, not customer adoption or forecasting training volume.

Ten fixture commodity categories: paddy, wheat, maize, onion, potato, tomato, cotton, soyabean, arhar, gram. Map to actual provider IDs explicitly. Presence in a fixture does not imply a trained forecasting model or calibrated storage assessment for that crop. Physical storage demonstration initially concerns paddy/wheat.

### 3.2 Storage-location correction without fabricated coverage

Retain the earlier ten-location/twenty-zone collection-centre fixture **only as a labelled synthetic load scenario**. Add a separate minimal integration scenario with a farmer-owned lot, one explicit farm-shed location, one monitored placement and one sensor node. These are separate profiles; do not quietly combine counts or claim ten real sheds cover 1,000 farmers.

Every lot needs an actual/fixture placement for route origin. A placement belongs to one location/zone at a time. Device assignment identifies what location the device monitors and its effective interval. A sensor at one bag/location does not establish the condition of all bags at all farms. Unmonitored lots may remain `UNKNOWN` and are still eligible for otherwise supported planning.

Farmer ownership, physical custody, transport coordination and who pays are distinct. Farm storage is user-provided context; transport roles/payment and actual FPO verification practice still need evidence.

### 3.3 Market archive and incremental updates

- Initial search window retained from the baseline: **2000-01-01 through 2026-10-12**. Then catch up through the latest completed India business date. On this review date, that is 2026-10-13; actual availability may lag.
- Acquire recent ten years first, then older available history. Do not hold initial integration hostage to finishing the national archive.
- Collect all permitted useful observations; select model training windows through evaluation. More old data is not automatically better for forecasting.
- Save source-supported partitions, bounded requests, checkpoints and the requested-versus-acquired coverage. Prefer supported bulk exports where available.
- At most three total attempts for safe transient read failures, respecting provider limits/Retry-After. Authorization, schema and CAPTCHA problems require correction; do not bypass access controls.
- Process active-FPO series ahead of background archive expansion. Record first-seen time, observed date, publication time if supplied, and revisions separately.
- A national catalogue count is not a historical row count. Earlier documents report 13 October access probes; those probes were not repeated in this review. Historical access and successful import remain acceptance dependencies.
- Exact clean rows, usable series, actual GB, and downloaded date coverage are **measured outputs**, not numbers we can declare in advance. Never generate missing dates to inflate them.

A market row retains source IDs/names, canonical market/commodity mappings, variety/grade or explicit missingness, observation date, modal/min/max price with units/currency, optional arrivals with units, source revision, receipt/batch identity and permitted original evidence.

Store authorized original exports/responses separately from normalized data. Use compressed partitioned Parquet for research datasets and PostgreSQL for operational/reference state. Large archives stay outside source-code Git. Dataset manifests list checksums, bytes, counts, gaps, rejected records, mappings and transformation versions.

### 3.4 Routes and transport are two different inputs

RouteEstimate describes origin/destination, distance, duration, travel mode, source, time and availability. It is not a rupee charge or proof of truck access.

TransportQuote describes exactly which origin/destination, load, date/validity, vehicle/availability and charges the quote covers. Missing charge information remains explicit. Preserve estimated expense separately from actual expense.

Before sharing trucks in an optimizer, specify trip grouping, load capacity, available trips, activation cost and how one trip charge is counted. **Do not assume a truck quote applies to any load or charge the same shared trip once per farmer.** Dedicated-trip synthetic cases are sufficient to start contract tests; real transport semantics are not thereby resolved.

### 3.5 Automatic hardware source

Current circuit: [wiring drawing](KisanNiti_Circuit/KisanNiti_Wiring.svg), [PNG](KisanNiti_Circuit/KisanNiti_Wiring.png), [KiCad source](KisanNiti_Circuit/KisanNiti_Circuit.kicad_sch).

| Sensor signal | ESP32 connection |
|---|---|
| VCC | 3V3 |
| GND | GND |
| SDA | GPIO21, with 4.7 kΩ pull-up to 3V3 |
| SCL | GPIO22, with 4.7 kΩ pull-up to 3V3 |

Use printed board/sensor signal labels; schematic symbol pin numbers are not physical header positions. The current probe reference is the protected SHT30 I2C cable probe, PROBOTS PRO6111, nominal 50 cm cable. Confirm the delivered item's wire identification. Keep supplied cable length for the initial build. The [supplier listing](https://probots.co.in/sht30-metal-waterproof-temperature-and-humidity-sensor-probe-50cm-cable-i2c.html) supports the I2C/3.3 V interface and two pull-ups; it does not validate our assembled system or grain-risk algorithm.

One-node assembly supplies carried forward: ESP32 board ×1; probe ×1; full-size breadboards ×2; male/male and male/female jumper packs; 4.7 kΩ resistors ×2; four-position 2.54 mm screw-terminal connector ×1; USB data cable; 5 V/2 A USB adapter; mounting box/base, cable ties and labels; small screwdriver and wire stripper; a labelled demonstration bag/container with about 1 kg paddy or wheat. No manual grain-moisture meter or its battery is required by the current design.

Probe at the demonstration grain, electronics outside the bag. Do not present the generic probe housing as food-contact certified. Keep demonstration grain separate. Electricity powers the board; a continuously available 2.4 GHz Wi-Fi connection/phone hotspot carries data. Mobile coverage alone is not an ESP32 cellular connection.

Required path:

```text
SHT30 → ESP32 firmware → Wi-Fi/hotspot → authenticated MQTT over TLS
      → ingestion adapter → SensorReadingV1 → Layer 2 checks
      → StorageConditionService → StorageAssessmentV1 → planning snapshot
```

No recurring manual sensor reading or spreadsheet entry. One-time wiring, firmware installation, Wi-Fi/device provisioning and assigning the probe to a monitored location are still necessary. The assistant can prepare software and diagnostics; the user performs physical assembly and device access steps.

Firmware framework, broker host, credentials, final topic/acknowledgement contract and buffering limits are not yet frozen. The minimum required behaviors are explicit identity, health, timestamps, duplicate handling, outage visibility and recoverable delivery. MQTT delivery confirmation alone must not be presented as proof of a committed backend record.

**Recommended contract-freeze choices, not historical decisions:** MQTT QoS 1 with deduplication; device-specific credentials/topic permissions; a persistent bounded local sample buffer and backend acceptance acknowledgement before deleting queued samples. Test behavior on buffer overflow and power loss, expose any data loss, and never promise unlimited offline history. If implementation selects a simpler buffer, report its limits explicitly and update acceptance criteria rather than silently claiming durability.

With no trusted clock after a reboot, retain device sequence/uptime and flag unknown measurement time. Do not stamp replayed data with upload time and call it fresh. Device assignment must be resolved at the measurement time where known; ambiguous history cannot be assigned to the current bag by guessing.

Simulator and hardware use the same consumer contract. Wokwi/custom simulation, if used, is an additional adapter test, not proof of electrical compatibility. No hardware simulator is a mandatory backend dependency.

## 4. Layer 2 — cleaning, validation and readiness

### 4.1 Sequence and ownership

1. **Layer 1 intake:** authenticate, bound payload size, parse safely, save permitted source evidence and ingestion outcome.
2. **Layer 2A cleaning:** parse known formats, map IDs, convert documented units, reconcile duplicates/corrections and represent missing values explicitly.
3. **Layer 2B validation:** check meaning, ranges, relationships, authority and invariants.
4. **Layer 2C readiness:** decide whether the version is usable for a named purpose under a named policy.

Initial cleaning uses deterministic rules. No ML cleaning model or automatic imputation model has been selected. Model-specific learned feature transformations belong to the forecasting pipeline and must be fitted using training-period data only.

### 4.2 Rules that must be implemented and independently tested

| Data | Rule | Failure handling |
|---|---|---|
| Identity | Versioned mappings to canonical IDs; scope matches FPO/member/lot | Quarantine ambiguous mapping; reject unauthorized cross-FPO access |
| Mass | Verified conversion; 1 quintal = 100 kg, 1 metric tonne = 1,000 kg | Unknown bag weight cannot become a guessed mass |
| Price | Explicit currency and denominator; distinguish total amount from rate | Unknown unit blocks that price use |
| Numbers | Exact inventory/money; finite measurements; explicit numeric policy | No NaN/Infinity; unsupported precision gets an issue, not hidden rounding |
| Dates | Source-specific date convention, UTC technical instants, FPO local dates | Ambiguous dates/timezones do not silently parse |
| Market consistency | Positive usable modal target; min ≤ modal ≤ max when present; arrivals ≥ 0 with unit | Preserve original; quarantine inconsistent target fields; optional missing arrivals need not block a price-only model |
| Missing market days | Missing stays missing | No invented zero or forward-filled value labelled observed |
| Duplicates/corrections | Source identity/revision or documented complete key/fingerprint | Identical replay has no duplicate effect; proven correction creates a version; unresolved conflict goes to review |
| Cross-source overlap | One report republished twice is not two independent observations | Preserve lineage; no silent averaging or double-counting |
| Sensor | Registered channel capabilities, finite values, RH 0–100%, time quality and assignment history | Invalid/offline/stale/unsupported assessment becomes UNKNOWN; no fabricated healthy reading |
| Route/quote | Correct origin/destination/load/date/currency and source status | Remove affected unavailable choice with reason; no zero-cost/zero-distance fallback |
| Inventory | Positive registered lot quantity; exact derived availability; valid owner and placement | No negative stock, double mutation or client-edited availability |

Kilograms and INR/kg are the existing **ENGINEERING LEAN**, not a fully frozen numeric schema. Before value-object implementation, freeze accepted precision, range, serialization and forecast-normalization rounding. Do not confuse canonical kg with a minimum planning increment. Actual-sale quantities must not be rounded to a planning increment.

### 4.3 Unusual prices are not automatically wrong

Retain the Layer 2 experimental past-only MAD screen only as a review flag: preceding 30 usable observations, at least 15, flag beyond 6 unscaled MAD; zero MAD/insufficient context means unassessable. These settings are not validated mandi-error thresholds.

A statistical flag alone does **not** delete, clip, replace or withhold an otherwise consistent price under the proposed development policy. Genuine spikes may be useful signal. Independent unit/identity/consistency errors can block a record. Confirmed corrections require evidence and history.

Measure false alarms and missed errors using reviewed real examples, including unflagged records and genuine spikes. Synthetic corruption tests demonstrate detection mechanics, not real-world detection accuracy. The [NIST outlier discussion](https://www.itl.nist.gov/div898/handbook/eda/section3/eda35h.htm) is methodological context, not validation of our 30/15/6 settings.

### 4.4 Three independent outputs

- **Quality:** PASS / REVIEW / FAIL for a specified record contract. PASS means software checks passed, not physical truth was certified.
- **Freshness facts:** measured/observed time, age, receipt time, explicit expiry and newer versions. Old history can be valid training data.
- **Readiness:** READY / READY_WITH_WARNINGS / BLOCKED for a specific purpose, policy and input version.

Each issue includes code, record/field, plain explanation, evidence, affected use, blocking scope, corrective action and rule version. A missing arrival can block an arrivals-dependent model but allow a price-only model. A missing sensor yields UNKNOWN storage and does not itself forbid allocation.

No universal TTL. The earlier proposed 15-minute stale setting for a five-minute demo sensor is an **ENGINEERING LEAN**, not a crop safety limit or general market-price expiry. Final boundary (`>` versus `≥`), clock handling and behavior must be frozen in the sensor policy tests.

### 4.5 Validation occurs again after cleaning

- OptimizationInputValidator checks normalized scope, evidence, quantities, policies and numeric feasibility before solving.
- PlanValidator recomputes hard validity and exact totals for optimizer, edited and manual plans. It neither calls providers nor proves optimality.
- Approval validation later uses current authority, availability and agreed freshness policy, not just the old snapshot.

## 5. Layer 3 — derive information, then allocate

### 5.1 Price forecasting

Forecast future **reported modal mandi price**, with declared commodity/market/variety/grade applicability, target date, currency and denominator. It is not a guaranteed farmer sale price.

Evaluation sequence carried forward:

1. Last usable observation and past-only moving-average baselines, with explicit age/window.
2. XGBoost with past lag/rolling features, calendar information and usable known arrivals; define horizon-specific targets for seven-date forecasts.
3. [Chronos-2](https://huggingface.co/amazon/chronos-2) as a general pretrained benchmark, not a verified mandi-specialist or our own training accomplishment.
4. A3T-GCN only if aligned multi-market history and a defensible graph justify the experiment. Geography/price similarity are hypotheses, not demonstrated trading links.

Use identical forecast origins/horizons and chronological rolling evaluation, plus an untouched later test period. Fit scalers/imputers/learned graph relationships within the applicable training period. Never use future actual prices or arrivals when forecasting earlier. Check pretrained overlap and state historical publication/revision limitations. [Forecasting: Principles and Practice](https://otexts.com/fpp3/toolbox.html) describes time-series evaluation using only observations available before each forecast origin.

Report MAE in the declared common unit, errors by crop/market/horizon, coverage/abstentions, spike performance, runtime and resource cost. Evaluate interval calibration if intervals are displayed. Compare on common targets and also report coverage; an accurate model that excludes difficult cases must not hide that exclusion.

The winner is evidence-selected. A forecast service uses one supported/pinned promoted model version per declared deployment configuration, not all four candidates sequentially for every request. Any fallback must be named, evaluated and tagged with its own model identity.

### 5.2 Storage-condition assessment

The SHT30 measures air temperature and relative humidity at its sensing location. Grain moisture at equilibrium depends on crop, temperature and surrounding humidity; instantaneous RH is not a direct measurement of the grain's moisture. [IRRI explains this equilibrium relationship](https://www.knowledgebank.irri.org/step-by-step-production/postharvest/drying/drying-basics/how-to-determine-the-emc?print=1&tmpl=component). The sensor's electrical/measurement capabilities are described in the [Sensirion SHT3x-DIS datasheet](https://sensirion.com/media/documents/213E6A3B/63A5A569/Datasheet_SHT3x_DIS.pdf).

Required output: HIGH / MEDIUM / LOW / UNKNOWN, supported scope, input-reading references, measurement window, freshness, policy version, rationale and evidence basis. Device health is a separate output.

**Still open:** the exact crop-specific rule, thresholds, exposure duration, hysteresis and validation dataset. No spoilage classifier has been trained, and no accurate probability or days-to-spoilage number is available.

There are two honest development paths:

- A labelled synthetic assessment fixture exercises HIGH/MEDIUM/LOW optimizer behavior and threshold-boundary tests. It cannot be presented as sensor-derived scientific accuracy.
- Real sensor readings pass through the full path, display automatic conditions, and produce UNKNOWN until an explicitly documented approximation policy is selected. That policy may be useful for a college demonstration, but must be labelled experimental and tested separately from claims about actual deterioration.

Do not silently adopt thresholds from the abandoned scratch prototype. Closing this approximation-policy task is required before claiming that the physical demo automatically converts conditions into meaningful crop-risk categories. Sensor integration itself need not wait for it.

### 5.3 Planning snapshot

The application gathers:

- FPO, commodity, selected lots/owners and participating quantities;
- current available-quantity snapshot and inventory versions;
- applicable origins, candidate mandis and allowed dates;
- usable forecasts and normalized effective lot/mandi/date price;
- routes, quotes and deterministic transport-cost specification;
- supported assessments or UNKNOWN;
- allocation, economic, fairness, storage, market-load, numeric and tie-break policies.

Freeze the evidence and policy versions into an evaluation basis. Prepare sparse admissible choices, retaining reasons for exclusions. The mathematical input references normalized facts, not raw provider payloads or database sessions.

### 5.4 Optimizer semantics

For each admissible lot/mandi/date choice, select a non-negative quantity that is an exact multiple of its declared planning increment.

```text
participating quantity > 0 and ≤ available snapshot
scheduled = sum of that lot's assignments
partial mode: scheduled ≤ participating
full mode: scheduled = participating
unallocated = participating − scheduled
```

Production scheduling increment is unresolved. Fixtures explicitly declare theirs; kg normalization does not select 1 kg scheduling. Full-mode incompatibility is explained, never silently converted to partial mode.

Priority order after hard technical/economic admissibility:

1. Maximize useful allocated quantity in partial mode.
2. Maximize the entire sorted vector of farmer fulfilment ratios (true leximin).
3. HIGH quantity then earliest-date quantities; repeat for MEDIUM, preserving prior optima. LOW has no additional urgency and UNKNOWN no invented rank.
4. Maximize expected net selling return.
5. Deterministic technical tie-break preserving all previous objectives.

Each farmer's fulfilment is scheduled quantity / participating quantity across **all their selected lots**. Multiple lots do not multiply fairness weight. Full mode can skip objectives already fixed by full allocation. Do not approximate these priorities with arbitrary weighted sums and call them equivalent.

Expected net selling return = predicted gross receipts − modeled transport charge. Report excluded charges; this is not full farming profit. Actual transport expense remains separate. Real economic guardrails and FPO acceptance of fairness/storage priorities are still required before production use.

### 5.5 Result and handoff to the manager

Return lot/farmer/mandi/date/quantity assignments, participating/allocated/remainder totals, price/cost arithmetic, exact fulfilment fractions, storage rationale, exclusions, search scope, evidence/policy/model versions and stage-by-stage proof information.

Keep distinct: input invalid; insufficient evidence; proven infeasible; zero useful allocation; feasible but not fully proven; technical failure/timeout. A timeout does not prove infeasibility. Feasible-but-incompletely-optimized proposal eligibility is an unresolved product policy; diagnostic display is not automatic approval eligibility.

Independently validate the candidate, then create an immutable evaluated proposal revision if eligible. No reservation, farmer instruction or sale follows merely from generating it. New evidence does not rewrite it. Manager changes create a new evaluated revision; changes after commitment normally start a new planning cycle from current truth.

## 6. One integration contract per boundary

The table below specifies required semantics; field names/serialization are an **ENGINEERING LEAN to freeze in work package W01**. It is not an already executable JSON Schema or database migration. Implementers must not each invent their own variants.

| Contract | Producer → consumer | Minimum contents and behavior |
|---|---|---|
| C01 SourceRecordEnvelopeV1 | Every adapter → intake/Layer 2 | Schema version, source, source identity/revision if available, dataset mode, real/synthetic origin, batch/event identity, raw evidence reference, observed/measured time or explicit missingness, received time; tenant scope where applicable |
| C02 LotInventorySnapshotV1 | Inventory + placement → planning | FPO, lot, owner, commodity/applicability, authoritative quantity, sold, remaining reserved, finalized adjustments, derived available, version, applicable placement; exact units |
| C03 MarketObservationV1 | Market adapter/Layer 2 → dataset/forecasting | Canonical series identity, business date, modal/min/max with units, optional arrivals, source/revision/first-known evidence and quality findings |
| C04 SensorReadingV1 | Simulator or real device adapter → Layer 2/storage | Registered device, boot/session ID, sequence/event ID, measured time/time-quality, temperature °C and RH %, per-channel validity, firmware/schema version, source origin, raw reference; backend receipt time and authoritative assignment resolution |
| C05 QualityReadinessV1 | Validators → all consumers | Input references, purpose, quality, freshness facts, readiness, issue codes/scopes, rules/policy versions and evaluation time |
| C06 RouteEstimateV1 | Routing adapter → planning/transport | Origin/destination, distance/duration units, mode, source/time/status and evidence; no cost field masquerading as provider truth |
| C07 TransportEconomicsV1 | Quote normalization → planning/optimizer | Quote references, load/trip/vehicle/date applicability, exact costs/basis, validity, missing charges, deterministic model version; shared-trip semantics explicitly disabled or specified |
| C08 PriceForecastV1 | Promoted forecasting adapter → planning | Series/applicability, target date, normalized rate, optional intervals, status, generated time, input cutoff/data version, model/features/numeric versions, provenance |
| C09 StorageAssessmentV1 | Storage service → planning | Covered location/lot applicability, category, reason, reading window/references, assessed time, freshness/time quality, policy/evidence basis; not a moisture value |
| C10 EvaluationBasisV1 | Orchestrator → replay/audit | Selected request, authorized scope, exact inventory/evidence/policy versions and values, resolved date/choice semantics, content identity; immutable |
| C11 OptimizationInputV1 | Orchestrator + input validator → pure engine | Ordered entities, sparse admissible choices, quantities/increments, owners, effective rates, transport math, storage categories, policies; separate execution config |
| C12 OptimizationResultV1 | Engine → independent validator | Sparse exact assignments, totals/remainders/fairness/economics, solution/feasibility/optimality/allocation/termination axes, objective proof prefix, diagnostics |
| C13 EvaluatedProposalV1 | Planning/PlanValidator → Layer 4 | Immutable revision and origin, evaluation basis, candidate/result where applicable, validation report, scope, summary and explicit eligibility; no reservations |
| C14 DatasetModelManifestV1 | Data/training jobs → evaluation/runtime | Counts/coverage/checksums/lineage/licensing; train/validation/test boundaries; feature/model versions and evaluation/promotion evidence; real/synthetic accounting |
| C15 ChangeJobV1 | Committed changed data → background work | Job ID/type, changed entity/data versions, dependency key, attempts, status, error, checkpoint and completion evidence; retry cannot duplicate a business effect |

### Contract-freeze checklist

For each contract: exact fields/types, required-versus-null rules, enum values, unit/precision/range, time semantics, identity/idempotency key, authorization, compatibility policy, valid example, invalid example, replay/correction example and producer/consumer tests.

Recommended serialization: canonical IDs as opaque strings; exact business quantities/rates as decimal strings with explicit units; UTC ISO-8601 timestamps; ISO local business dates; `null` plus reason for unknown values. Do not use binary floats for stock or money. Sensor measurement precision is independent of stock precision. Final accepted scales/magnitudes must be decided before C02/C07/C08/C11 implementation.

Sensor tenant/assignment authority comes from registered device credentials and backend history, not untrusted `fpo_id` or `zone_id` in a payload. Real-versus-synthetic classification also comes from the registered source and execution mode, not a user-editable label alone. Clock-unknown samples remain recorded but are not fresh trusted risk evidence.

Global market/reference data can be reused across FPOs; private inventory, quotes, assessments, plans and device assignments require tenant protection. A contract version change requires coordinated producer/consumer changes and migration/replay checks.

## 7. When new data arrives

| Trigger | Required work | Work that does not happen automatically |
|---|---|---|
| Market batch changed | Validate/version; coalesce changed dependencies; infer supported forecasts with current model | Training per row; re-solving every stored plan |
| Duplicate market batch | Record/check replay outcome; no duplicate derived effect | New forecast merely because the scheduler fired |
| Sensor reading | Validate/deduplicate; resolve assignment/time; update supported assessment | Price-model training; sale; silent revision overwrite |
| Inventory event | Authorized transaction; update derived availability/version | Automatic market-price adjustment |
| New quote/route | Validate/store applicability; make changed evidence visible | Treat unknown old validity as unlimited validity |
| Manager plan/replan | Gather/freeze usable inputs; solve; independently validate | Provider calls from inside the optimizer |
| Model candidate evaluated | Explicit promotion only after evaluation; record version and rollback | Silent deployment on a timer |

Training starts as a deliberate offline evaluated run. Retraining cadence is selected after data/accuracy/drift and resource measurements. Forecast updates follow relevant successful batches. Optimization starts on a planning request. Dependency jobs must be durable/recoverable and deduplicated by input version; failure after DB commit must not lose required downstream work.

A replay reconstructs the original snapshot from stored evidence. A replan uses current facts. They are different operations.

## 8. Demo, live and research must stay distinguishable

**DEMO** may combine real public prices, fictional organization records, synthetic routes/quotes and either simulated or real sensor measurements. Label the whole proposal as a demonstration and retain the origin of every input. Disable external farmer communication for fictional identities.

**RESEARCH** training/evaluation uses actual eligible market observations. Synthetic rows do not count toward real forecast accuracy or our own training-data claims. Pretrained model use does not mean we trained its upstream corpus.

**LIVE/PILOT** requires real authority, inventory, allowed evidence, enabled policies and the corresponding acceptance gates. Missing contracts/policies cannot silently fall back to demo settings.

A disconnected physical device should make its assessment UNKNOWN with reasons while independent market/route/stock work continues. Required unavailable forecast or transport evidence removes the affected choice. If no useful choices remain, explain insufficient evidence; do not label a provider outage a proven impossible business plan.

## 9. Remaining decisions and evidence — do not conceal these

| ID | Open item | Can proceed now | What closes it |
|---|---|---|---|
| G01 | Repository blueprint gate and conflicting old hardware references | This design handoff and explicit amendment proposal | Record latest user decisions, freeze affected contracts, explicitly update coding status before product implementation |
| G02 | Exact schema/API/numeric policy, package versions | Semantic contract review/examples | W01 exact schemas, quantity/rate scales, rounding/ranges, enum/error/idempotency definitions, lockfile and compatibility tests |
| G03 | Authorized real historical source access and actual coverage | Fixture adapters, manifests, bounded access setup | Successful authorized sample import + terms/access evidence, then measured acquisition manifest |
| G04 | Storage approximation policy | Automatic telemetry, UNKNOWN, synthetic-category integration | Named crop-specific rule with sources, applicability, durations/thresholds, version, tests and honest experimental label; field calibration for field claims |
| G05 | Physical node, firmware, broker and outage behavior | Simulator/contract and software design | Exact toolchain/driver pinning, compile, bench records, durable/replay tests and real network reachability |
| G06 | Operational origin/transport/quantity increment | Explicit synthetic scenario policies; generic typed boundaries | Per-FPO actual placement/dispatch, quote/load/grouping/arranger/payer/availability rules and increment evidence |
| G07 | Economic guardrail and accepted fairness/storage trade-offs | Mathematical fixtures and oracle tests | Representative accepted/rejected scenarios and versioned policy; no unvalidated arbitrary thresholds |
| G08 | Forecast winner and supported series/horizons | Baselines/evaluation harness after real-data access | Held-out results, coverage, cost/latency, provenance and promotion report |
| G09 | Solver backend, exact tie-break, timeout eligibility | Reference oracle and prototype design | Full objective correctness, numeric reconstruction, realistic benchmarks and explicit result eligibility policy |
| G10 | Market calendars/date feasibility and freshness actions | Explicit fixture dates; no same-day assumption | Actual operational rules and evidence-specific warning/block behavior |
| G11 | Layers 4–5 authority, farmer acceptance, off-plan reconciliation | Preserve boundaries and future acceptance scenarios | Separate layer review and real workflow decisions; no invented reservation-release order |
| G12 | Deployment, recovery and performance limits | Modular architecture and measurable tests | Chosen reachable endpoint/secret handling, backup restoration, workload evidence and documented limits |

These are not all questions the user must personally answer. Agents can settle engineering details and run experiments; real access, physical checks and FPO-policy claims require their appropriate evidence. No honest design can decide an unknown dataset size or model accuracy into existence.

For college demo policy, each necessary assumption must be explicit, versioned and confined to DEMO. The existing authorization allows labelled dummy data; it does not turn dummy values into validated FPO policy. No production threshold is invented by this handoff.

## 10. Implementation work packages and order

Each task must follow the repository task template: scope, contract, exclusions, tests, completion evidence. Finish the relevant prerequisite, then implement a small auditable piece. Do not assign independent agents overlapping ownership of the same contract.

| Task | Deliverable | Depends on | Stop/exit condition |
|---|---|---|---|
| W00 | Promote design amendments/status accurately; reconcile hardware/location supersession | This review | Authoritative status and decision register agree; no silent production unknown closure |
| W01 | Freeze contract examples/schemas, numeric/date semantics, demo policy bundle and versioned configuration | W00 | Producers/consumers have one agreed contract; unresolved production policies remain gated |
| W02 | Foundation: tenant boundary, exact values, migrations, audit and job/idempotency primitives | W01 + coding authorization | Exact-value, authorization, transaction and recovery tests pass |
| W03 | Seeded organization/storage fixtures and inventory/placement core | W02 | Counts, ownership, history and stock invariants pass; no unfinished off-plan workflow invented |
| W04 | Layer 1 market adapter + bounded authorized import + manifests | W01–W02; G03 for real mode | Source contract and replay pass; actual coverage/access failures reported |
| W05 | Layer 2 normalization, quality, readiness and dataset releases | W03–W04 contracts | Cleaning test corpus and lineage/rebuild checks pass |
| W06 | Simulator and sensor ingestion/storage interface | W01–W03 | Same schema, auth, dedup, assignment, time and UNKNOWN tests pass without physical device |
| W07 | Forecast baselines, candidate evaluation and normalized inference | W04–W05 | Leakage checks, measured evaluation and pinned model artifact; no unearned winner |
| W08 | Route/quote adapters and deterministic transport math | W01/W03/W05 | Applicability/unit/cost/failure contract tests pass; real mode separately gated |
| W09 | Pure optimizer oracle, staged formulation and independent PlanValidator | W01/W03/W05/W07/W08 | Exact complete objective comparisons, numerical and failure tests pass |
| W10 | Planning orchestration, frozen snapshot, immutable proposal and replay | W06–W09 | Layer 1→2→3 golden/failure paths pass against actual PostgreSQL |
| W11 | Real ESP32 firmware, provisioning and hardware adapter acceptance | W06 + G05; hardware available | Real readings and outages/reboot/late-upload tests pass; risk policy separately verified |
| W12 | Layer 4–5 implementation and full application acceptance | Separate layer decisions, W10 and relevant real gates | Approval, communication, partial sale, reconciliation and replan all verified |

Start with one small complete Layer 1→2→3 scenario while scaling archive collection independently. The 30,000-lot stress dataset is not one mandatory solver request. Measure realistic per-FPO planning size and report limits before increasing it.

An agent working later must read this handoff, the acceptance plan and repository rules; identify its work package; state decisions versus open dependencies; run its required checks; and report actual files, tested versions, pass/fail/blocked results and remaining limits. It must not call the whole product complete because its own module passes.

## 11. Evidence and maintenance

Every run should be traceable by request/batch/job/run ID, tenant scope and input versions. Logs must reveal failure causes without publishing private farmer data or credentials. Keep dependency locks, migrations, configuration examples, schema examples, source mappings and test fixtures under version control; keep secrets/large datasets out.

Repair guides must cover source authentication/schema change, mapping/unit conflict, interrupted import, missing forecast, disconnected sensor/clock error, invalid quote, failed solver, rejected validation and backup restoration. Preserve originals/evaluation snapshots so a repaired adapter can replay inputs without duplicate stock effects.

**Review status:** source documents reconciled; latest hardware/location amendments documented; integration contracts, open gates and required tests specified. No production software tests, electrical tests or field validation were performed in this review. See the [review record](KisanNiti_Layers_1_to_3_Review_Record.md) for exactly what was checked.
