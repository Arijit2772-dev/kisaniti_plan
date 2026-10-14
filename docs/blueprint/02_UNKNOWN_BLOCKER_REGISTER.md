# `UNKNOWN` / Blocker Register

> **15 October 2026 context:** all 21 production unknown entries remain open. User target context now includes farm-shed bag storage and ESP32 + SHT30 automatic monitoring. DHT22 references below are historical examples; see [amendments](../architecture/2026-10-15_Conversation_Amendments.md). This does not validate real custody, thresholds, authority or reconciliation. Desk-research approximations and DEMO assumptions for all 21 entries (Patiala): [research note](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md); statuses remain OPEN.

This register indexes unresolved questions from the [consolidated architecture handoff](../architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md). All entries remain unresolved. They must not be converted into implementation behavior without an explicit approved decision and appropriate evidence.

## A. Core business contract blockers

### CORE-001 — Physical produce custody/location before sale

**Unknown ID:** CORE-001<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER`<br>
**Question:** Where does produce physically remain before sale?<br>
**Why it matters:** Custody and location affect authoritative placement, dispatch origin, storage assessment, routing, transport economics, and operational execution.<br>
**Affected future components:** ProduceLot placement, storage, routing, transport cost, planning, approval, communication, execution.<br>
**Current architecture constraints:** A ProduceLot has one active physical placement at an instant, with placement history; produce origin must not be invented.<br>
**What must not be assumed:** Do not assume produce remains at the farmer, collection center, warehouse, or any other location.<br>
**Required evidence:** Real FPO workflow evidence describing custody and physical movement before sale.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§15, 63.1, 66.1, 68.

### CORE-002 — Execution path to mandi

**Unknown ID:** CORE-002<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER`<br>
**Question:** Is the execution path farmer→mandi or warehouse/collection-center→mandi, or does it vary?<br>
**Why it matters:** The physical execution path determines route origins, transport coordination, instructions, and the relationship between storage and sale execution.<br>
**Affected future components:** Storage/placement, routing, transport cost, SellingAssignment execution, communication, audit.<br>
**Current architecture constraints:** Routing is provider-independent; transport economics are separate; planned and actual execution must remain distinct.<br>
**What must not be assumed:** Do not hard-code a farmer-origin or collection/warehouse-origin path.<br>
**Required evidence:** Real FPO workflow evidence for normal and exceptional dispatch paths.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§19–20, 53–55, 63.2, 68.

### CORE-003 — Transport arranger

**Unknown ID:** CORE-003<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER`<br>
**Question:** Who arranges transport?<br>
**Why it matters:** Ownership of transport coordination affects workflow, authority, communication, quotes, grouping, and failure handling.<br>
**Affected future components:** TransportCostService, planning, approval, communication, execution, authorization, audit.<br>
**Current architecture constraints:** Transport economics remain separate from routing; V1 is not fleet management/VRP unless workflow evidence requires it.<br>
**What must not be assumed:** Do not assume the farmer, FPO, buyer, mandi, or a platform arranges transport.<br>
**Required evidence:** Real FPO workflow evidence from those who arrange and approve transport.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§20, 63.3, 66.3, 68.

### CORE-004 — Actual transport pricing

**Unknown ID:** CORE-004<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER`<br>
**Question:** How is transport actually priced, including any grouping, toll, rate, vehicle, quantity, or quote semantics?<br>
**Why it matters:** The answer defines valid transport economics and determines what can be frozen for planning and later compared with actual expense.<br>
**Affected future components:** TransportCostModel, TransportCostService, OptimizationInput, EconomicPolicy, revision evaluation basis, ActualSale/expense feedback.<br>
**Current architecture constraints:** The design must remain future-capable for fixed + variable structure, but no formula is frozen; estimated cost and actual expense remain separate.<br>
**What must not be assumed:** Do not invent a transport formula, grouping rule, payer, quote-validity rule, or arbitrary Big-M.<br>
**Required evidence:** Actual FPO/transporter pricing records, quotes, invoices, and workflow validation.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§20, 33, 63.4, 66.4, 68.

### CORE-005 — Economic acceptability

**Unknown ID:** CORE-005<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER`<br>
**Question:** What exact rule makes a candidate or plan economically acceptable, and what manager override semantics apply?<br>
**Why it matters:** EconomicPolicy defines the allowed region and provides the guardrail for allocation, fairness, storage urgency, and expected net selling return.<br>
**Affected future components:** EconomicPolicy, technical/economic admissibility, optimizer, PlanValidator, approval, manager overrides.<br>
**Current architecture constraints:** The policy must be versioned and deterministic; it may have choice-, farmer-, and plan-level requirements; negative expected net return is not automatically forbidden.<br>
**What must not be assumed:** Do not invent a minimum price, threshold, loss rule, override rule, or treat expected net selling return as profit.<br>
**Required evidence:** Real FPO decision records and manager validation of acceptable/unacceptable scenarios.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§32–33, 37–39, 63.5, 66.5, 68.

### CORE-006 — Minimum optimizer scheduling increment

**Unknown ID:** CORE-006<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER`<br>
**Question:** What is the minimum schedulable allocation increment, and is it FPO-, commodity-, or lot-dependent?<br>
**Why it matters:** The planning increment constrains legal optimizer allocations and full-mode compatibility.<br>
**Affected future components:** Quantity value objects, AllocationPolicy, OptimizationInput, optimizer, PlanValidator, UI quantities.<br>
**Current architecture constraints:** Canonical quantity unit, planning increment, and solver normalization quantum are distinct; exact arithmetic is required; ActualSale is not constrained by planning increment.<br>
**What must not be assumed:** Do not select 1 kg, 10 kg, 1 quintal, or any other increment.<br>
**Required evidence:** Real FPO scheduling/dispatch practice across relevant commodities and lot sizes.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§9, 29–30, 41, 63.6, 66.6, 68.

### CORE-007 — Quality/variety relevance

**Unknown ID:** CORE-007<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER`<br>
**Question:** Does quality or variety materially affect mandi eligibility or effective price?<br>
**Why it matters:** Validated quality/variety effects may alter canonical data, technical admissibility, and lot-level effective economics.<br>
**Affected future components:** ProduceLot, Commodity mapping, PriceForecast normalization, effective optimizer price, PlanValidator.<br>
**Current architecture constraints:** Variety/grade remains extensible; domain PriceForecast remains indexed by mandi/commodity/date; optimizer can accept effective `p[l,m,d]`.<br>
**What must not be assumed:** Do not assume quality is irrelevant, and do not fabricate eligibility or price adjustments.<br>
**Required evidence:** Real FPO/mandi transaction evidence plus appropriate domain/market evidence.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§18, 23, 63.7, 66.7, 68.

### CORE-008 — MarketLoadPolicy production semantics/calibration

**Unknown ID:** CORE-008<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER`<br>
**Question:** What production MarketLoadPolicy mode and calibration/window apply, especially if `STRICT` is considered?<br>
**Why it matters:** `STRICT` changes feasibility, while `ADVISORY` and `DISABLED` do not; empirical arrivals are not official physical capacity.<br>
**Affected future components:** MarketLoadPolicy, OptimizationInput, optimizer, PlanValidator, revision evaluation basis, UI warnings.<br>
**Current architecture constraints:** Modes are `STRICT`, `ADVISORY`, and `DISABLED`; advisory has no feasibility effect or optimization penalty; provenance must be frozen; strict missing values never become zero or infinity.<br>
**What must not be assumed:** Do not treat historical arrivals as physical capacity, enable `STRICT`, or choose a calibration/window without validation.<br>
**Required evidence:** Real FPO operating practice, authoritative mandi context where available, and stronger empirical validation before `STRICT` use.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §31, §§63.8, 66.8, 67, 68.

### CORE-009 — Off-plan ActualSale and active-reservation reconciliation

**Unknown ID:** CORE-009<br>
**Classification:** `UNKNOWN / CORE BUSINESS CONTRACT BLOCKER; MUST VALIDATE BEFORE PILOT`<br>
**Question:** How must an off-plan ActualSale reconcile with active reservations on the same ProduceLot?<br>
**Why it matters:** Reality must remain recordable without breaking inventory invariants or silently stealing quantity from an arbitrary approved commitment.<br>
**Affected future components:** InventoryService `record_sale()`, QuantityReservation, ActualSale, cancellation/release, deviation workflow, authorization, audit.<br>
**Current architecture constraints:** Transactionally preserve `actual sold + remaining active reservations + other finalized decrements <= authoritative lot quantity`; linked sales consume corresponding reservation; unlinked sales must not be rejected merely because a plan exists or silently choose a reservation.<br>
**What must not be assumed:** Do not choose reservation release order, a manager reconciliation flow, or any other off-plan behavior.<br>
**Required evidence:** Real FPO handling of off-plan sales after commitment, including authority and audit expectations.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§14, 54–55, 62, 63.9, 66.11, 68.

## B. Must validate before pilot

### PILOT-001 — Produce registration workflow

**Unknown ID:** PILOT-001<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** What is the real produce registration workflow?<br>
**Why it matters:** Registration establishes authoritative lot facts and their audit trail.<br>
**Affected future components:** FarmerMember, ProduceLot, inventory intake, authorization, audit, UI.<br>
**Current architecture constraints:** Produce belongs in ProduceLots rather than directly on Farmer; one farmer may have multiple lots/commodities; facts and derived information remain separate.<br>
**What must not be assumed:** Do not invent intake steps, required fields, statuses, approvals, or custody transitions.<br>
**Required evidence:** Observation/interviews and real intake records from the FPO.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§7–8, 63–64.1.

### PILOT-002 — Quantity/quality verification authority

**Unknown ID:** PILOT-002<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** Who verifies registered quantity and quality, and with what authority?<br>
**Why it matters:** Verification determines which values are authoritative and who may establish or correct them.<br>
**Affected future components:** ProduceLot intake, quantity adjustments, quality/variety data, authorization, audit.<br>
**Current architecture constraints:** Inventory truth uses exact quantities and finalized adjustments; quality effects remain unknown; corrections must not silently delete history.<br>
**What must not be assumed:** Do not assign verification to a farmer, manager, warehouse staff member, or external actor without validation.<br>
**Required evidence:** Real FPO roles, standard operating practice, and sample verification records.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§7, 9, 11, 64.2.

### PILOT-003 — Staff roles and authority

**Unknown ID:** PILOT-003<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** What exact manager/staff roles, permissions, and authorities exist?<br>
**Why it matters:** Authorization must enforce tenant isolation and real operational responsibility.<br>
**Affected future components:** Identity, authorization, approval, inventory, storage, sales, transport, audit.<br>
**Current architecture constraints:** Primary web actor is FPO manager/staff; authorization must remain flexible and server-side; privileges must not be encoded from email/name.<br>
**What must not be assumed:** Do not invent manager, CEO, operator, warehouse, accountant, acceptance, or sale-recording privileges.<br>
**Required evidence:** Real FPO organizational roles and approval/recording authority.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§6–7, 47, 49, 64.3.

### PILOT-004 — Farmer acceptance requirement

**Unknown ID:** PILOT-004<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** Must a farmer explicitly accept an approved instruction before execution, and who may record that acceptance?<br>
**Why it matters:** Acceptance affects commitment, communication, authority, cancellation, and execution workflow.<br>
**Affected future components:** Approval, reservation, communication, execution, authorization, audit/feedback.<br>
**Current architecture constraints:** The manager remains in control; the system does not autonomously sell; approved instructions may be communicated to farmers.<br>
**What must not be assumed:** Do not require, omit, or assign farmer consent behavior without validation.<br>
**Required evidence:** Real FPO decision and farmer communication practice.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§2, 7, 52, 64.4, 66.9, 68.

### PILOT-005 — ActualSale recording authority

**Unknown ID:** PILOT-005<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** Who is authorized to record an ActualSale?<br>
**Why it matters:** ActualSale mutates inventory truth and may consume reservations or trigger reconciliation.<br>
**Affected future components:** ActualSale, InventoryService, authorization, idempotency, audit, feedback.<br>
**Current architecture constraints:** ActualSale belongs to ProduceLot; assignment link is optional; reality must remain separate from the approved plan; recording must preserve inventory invariants.<br>
**What must not be assumed:** Do not assign sale recording to a manager, farmer, accountant, warehouse actor, or automated source without evidence.<br>
**Required evidence:** Real FPO sale-closing and record-keeping workflow.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§14, 53–55, 59, 64.5.

### PILOT-006 — Actual transport expense recording

**Unknown ID:** PILOT-006<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** Is actual transport expense reliably recorded, by whom, and from what evidence?<br>
**Why it matters:** Actual expense cannot be assumed from modeled cost and is needed for operational reconciliation and evaluation.<br>
**Affected future components:** Transport feedback/records, ActualSale context, audit, analytics, economic evaluation.<br>
**Current architecture constraints:** Modeled transport cost and actual transport expense must remain separate.<br>
**What must not be assumed:** Do not infer actual expense from route or estimated transport cost, or invent recording ownership.<br>
**Required evidence:** Real invoices, payment/expense records, and the FPO recording workflow.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§3, 20, 33, 64.6.

### PILOT-007 — Storage thresholds

**Unknown ID:** PILOT-007<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** What scientifically and operationally validated storage thresholds produce `HIGH`, `MEDIUM`, `LOW`, or `UNKNOWN` assessments?<br>
**Why it matters:** StorageAssessment affects urgency, warnings, and optimizer priorities.<br>
**Affected future components:** StorageConditionService, StorageAssessment, policy versions, optimizer, warnings, tests.<br>
**Current architecture constraints:** DHT22 measures ambient temperature and relative humidity, not grain moisture/quality/spoilage; `UNKNOWN` is mandatory; missing/stale/invalid/offline never becomes `LOW`.<br>
**What must not be assumed:** Do not invent thresholds or claim direct spoilage prediction.<br>
**Required evidence:** Appropriate agricultural/scientific authorities or research plus validation in the actual storage context.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§16, 37, 64.7, 67.

### PILOT-008 — IoT deployment details

**Unknown ID:** PILOT-008<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** What sampling rate, MQTT QoS/topic/authentication, device deployment, and operational urgency semantics apply?<br>
**Why it matters:** These details determine trustworthy ingestion, device health, freshness, security, and assessment interpretation.<br>
**Affected future components:** ESP32/DHT22 deployment, MQTT ingestion, DeviceAssignment, SensorReading, StorageConditionService, observability/security.<br>
**Current architecture constraints:** Device assignment has history; device health is distinct from storage condition; the provider normalization/failure pattern applies.<br>
**What must not be assumed:** Do not select sampling frequency, QoS, topic structure, authentication, sensor placement, or urgency behavior.<br>
**Required evidence:** Real deployment constraints, security requirements, device tests, and FPO operational validation.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§15–16, 21, 64.8.

### PILOT-009 — Fairness policy acceptance

**Unknown ID:** PILOT-009<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** Do managers accept the current-run per-farmer true leximin fulfilment policy?<br>
**Why it matters:** Fairness materially affects allocation ordering and manager acceptance of recommendations.<br>
**Affected future components:** FairnessPolicy, optimizer, OptimizationResult, UI explanations, feedback/analytics.<br>
**Current architecture constraints:** Current V1 design is per-farmer true leximin among participating farmers with positive participating quantity; it must remain versioned and not be hard-coded across the application.<br>
**What must not be assumed:** Do not treat current fairness semantics as accepted real-FPO practice without pilot validation.<br>
**Required evidence:** Scenario-based validation and decisions from real FPO managers.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§34–36, 64.9.

### PILOT-010 — Storage versus economic priority

**Unknown ID:** PILOT-010<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** How do real FPO managers trade storage urgency against economic outcomes?<br>
**Why it matters:** The current objective hierarchy puts storage before expected net selling return, but requires an economic guardrail or revision of that hierarchy.<br>
**Affected future components:** EconomicPolicy, StoragePolicy, optimizer objective hierarchy, PlanValidator, UI explanation.<br>
**Current architecture constraints:** Do not blend rupees, fairness, and urgency with arbitrary weights; do not allow storage urgency to blindly create enormous economic sacrifice.<br>
**What must not be assumed:** Do not invent trade-off weights, thresholds, or override behavior.<br>
**Required evidence:** Real manager decisions over representative storage/price scenarios.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§37–39, 64.10.

### PILOT-011 — Freshness warning/block rules

**Unknown ID:** PILOT-011<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** Which freshness changes warn, block approval, expire a revision, or require refresh/replan?<br>
**Why it matters:** Approval must distinguish historical validity, current commit validity, and freshness without silently rewriting frozen revisions.<br>
**Affected future components:** ApprovalFreshnessPolicy, ApprovalContext, forecast/storage/route/transport/policy evidence, UI, audit.<br>
**Current architecture constraints:** No universal TTL; freshness detection and approval action are separate; blocking requires active policy; provider calls must not occur while inventory locks are held.<br>
**What must not be assumed:** Do not invent TTLs, materiality thresholds, warning rules, blocking rules, or auto-replan behavior.<br>
**Required evidence:** Real FPO approval practice, evidence-source behavior, and explicit policy decisions.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§48–49, 64.11, 68.

### PILOT-012 — Same-day operational cutoffs

**Unknown ID:** PILOT-012<br>
**Classification:** `UNKNOWN / MUST VALIDATE BEFORE PILOT`<br>
**Question:** What same-day sale, approval, communication, or dispatch cutoff rules apply in the FPO's local timezone?<br>
**Why it matters:** Cutoffs affect date validity, approval, communication, and execution feasibility.<br>
**Affected future components:** Time value objects, Planning/Approval validators, communication, execution, FPO policy.<br>
**Current architecture constraints:** Technical timestamps use UTC; each FPO has an explicit local timezone; naive timestamps are prohibited.<br>
**What must not be assumed:** Do not invent cutoff times, date rollover rules, or exceptions.<br>
**Required evidence:** Real FPO daily operations and mandi/transport timing constraints.<br>
**Status:** OPEN<br>
**Resolution:** Unresolved.<br>
**Sources:** Architecture §§47, 60, 64.12.

## Register count

- Core business contract blockers: **9**.
- Must-validate-before-pilot entries: **12**.

`CORE-009` is also explicitly a must-validate-before-pilot gap in the architecture; it is counted once in the core list and cross-classified rather than duplicated.
