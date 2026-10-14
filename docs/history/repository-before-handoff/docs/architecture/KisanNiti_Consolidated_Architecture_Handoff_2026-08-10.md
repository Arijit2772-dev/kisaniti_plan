# KisanNiti — Consolidated Architecture Handoff
## Final pre-implementation source of truth
**Consolidated:** 2026-08-10

> **Use this file as the current source of truth for KisanNiti architecture.**
> It supersedes the earlier architecture handoff where this file explicitly refines or corrects it.
>
> In a new chat, say:
>
> **“Continue KisanNiti from this consolidated handoff. Treat it as the current source of truth. Do not silently revive revised/rejected ideas. Keep classifying claims as CONFIRMED, DESIGN DECISION, ENGINEERING LEAN, or UNKNOWN. If new real-FPO evidence conflicts with this handoff, flag the conflict explicitly before changing anything. Do not invent unresolved business workflow. Implement only already-frozen contracts and keep each coding task small, tested, and auditable.”**

---

# 1. Decision labels

Use these labels consistently:

- **CONFIRMED** — supported by project/source material or directly validated.
- **DESIGN DECISION** — intentional KisanNiti product/architecture decision.
- **ENGINEERING LEAN** — preferred technical direction, but benchmark/validation is still required.
- **UNKNOWN** — must be validated; do not invent.

When making a new decision, cross-check it against:

1. FPO-manager workflow.
2. Inventory correctness.
3. Forecasting/storage/routing/transport integration.
4. Reliability and failure semantics.
5. Multi-FPO isolation.
6. Auditability/debuggability.
7. Future extensibility.
8. Concurrency/idempotency.
9. Existing revised/rejected decisions.
10. Real-world execution rather than only the optimizer.

---

# 2. Product definition

## CONFIRMED

KisanNiti is a **decision-support system for FPO managers**.

It combines:

- farmer/FPO produce availability,
- mandi/market information,
- price forecasts,
- storage condition/urgency,
- routing information,
- transport economics,
- empirical market-load information,
- fairness/business policy,

to propose a coordinated:

```text
farmer / ProduceLot
        ↓
mandi
        ↓
selling date
        ↓
quantity
```

plan.

The manager remains in control.

KisanNiti does **not** autonomously sell produce.

Approved instructions may be communicated to farmers.

---

# 3. Fundamental architecture rule

## DESIGN DECISION — strongest system-wide invariant

Always separate:

```text
RAW / AUTHORITATIVE FACTS
        ↓
DERIVED INFORMATION
        ↓
SYSTEM DECISION
        ↓
HUMAN DECISION
        ↓
REALITY
```

Examples:

```text
MarketObservation        ≠ PriceForecast
SensorReading            ≠ StorageAssessment
RouteEstimate            ≠ TransportCostEstimate
optimizer proposal       ≠ approved plan
approved plan            ≠ ActualSale
estimated transport cost ≠ actual transport expense
```

Never overwrite one layer with another.

This principle is central to debugging, auditability, and future model replacement.

---

# 4. Application shape

## DESIGN DECISION

V1 is a **modular monolith**, not microservices.

```text
Next.js frontend
       ↕
FastAPI modular backend
       ↕
PostgreSQL
```

Logical modules/interfaces may include:

- Identity / authorization
- FPO / members
- Inventory
- Storage
- IoT ingestion
- Forecasting
- Routing
- Transport cost
- Planning
- Optimization
- Validation
- Communication
- Feedback
- Audit

Do not introduce Kafka, Kubernetes, Redis, separate microservices, etc. unless justified later.

One PostgreSQL database is sufficient initially.

PostGIS/TimescaleDB may be added only if demonstrated useful.

---

# 5. Dependency direction

## DESIGN DECISION

Prefer code dependencies to follow conceptual data flow:

```text
provider adapters
      ↓
normalized domain services
      ↓
planning / orchestration
      ↓
OptimizationInput
      ↓
pure OptimizationEngine
      ↓
OptimizationResult
      ↓
validation / proposal
```

Avoid:

- optimizer importing FastAPI/PostgreSQL/provider SDKs,
- TransportCostService exposing Google-specific objects,
- circular module ownership,
- domain logic living in controllers,
- provider strings being used as cross-module identity.

Stable domain interfaces are the extension points.

---

# 6. Multi-FPO

## DESIGN DECISION

Support multiple FPOs from the beginning.

All tenant-owned business objects must be unambiguously FPO-scoped, including:

- FarmerMember,
- staff,
- ProduceLot,
- StorageLocation,
- SellingPlan,
- QuantityReservation,
- ActualSale,
- feedback,
- configuration/policies.

Server-side authorization must enforce FPO isolation.

Do not hard-code a demo FPO.

### Refinement

`Mandi` may be global/reference master data rather than FPO-owned.  
FPO-specific mandi selection/configuration remains tenant-scoped.

Do not mechanically translate the conceptual graph into DB tables before relationship/invariant review.

---

# 7. Actors

## Current direction

Primary operational web actor:

- FPO manager/staff.

Farmer:

- FPO member,
- not an employee by default,
- primarily represented by FPO staff in V1,
- receives approved instructions,
- future self-declaration/querying may be added.

## UNKNOWN

Exact roles/permissions:

- manager,
- CEO,
- operator,
- warehouse staff,
- accountant,
- farmer acceptance authority,
- sale-recording authority.

Authorization must be flexible enough to map validated roles later.

Do not encode privileges from email/name.

---

# 8. Farmer and ProduceLot

## DESIGN DECISION

Do not store crop and available quantity directly on Farmer.

Use:

```text
FPO
 └── FarmerMember
      └── ProduceLot(s)
```

A `ProduceLot` is:

> a tracked quantity of one commodity belonging to one farmer/member and treated as one inventory batch.

One farmer may have multiple lots and commodities.

One physical lot can later be physically split into child/sub-lots if necessary.

Selling across multiple mandis/days does **not** physically split the lot; use multiple SellingAssignments.

---

# 9. Quantity representation

## DESIGN DECISION

Use one canonical internal quantity unit with **exact arithmetic**.

Likely base unit is kilograms, but this is **not fully frozen**.

UI may display quintals.

Never use ordinary binary floating-point as inventory truth.

### Distinguish three concepts

1. **Canonical quantity unit** — domain representation.
2. **Planning increment** `Δ_l` — legal optimizer scheduling increment.
3. **Solver normalization quantum** — internal exact scaling chosen for mathematical representation.

These are not the same thing.

### UNKNOWN / BLOCKER

Minimum schedulable allocation increment:

- 1 kg?
- 10 kg?
- 1 quintal?
- FPO/commodity/lot dependent?

ActualSale quantities are reality and are **not** constrained by planning increment.

---

# 10. Money and rate representation

## DESIGN DECISION

Money and transport economics use exact decimal/integer-minor-unit arithmetic.

Do not use ordinary binary float as business money truth.

Distinguish:

```text
Money
```

from:

```text
Rate
(e.g. ₹/kg, ₹/quintal, ₹/km, ₹/tonne-km)
```

Raw ML floating predictions are normalized through an explicit deterministic, versioned numeric policy before becoming planning `PriceForecast` values.

No hidden intermediate business rounding.

Display rounding is separate from authoritative values.

---

# 11. Inventory architecture

## DESIGN DECISION

Inventory truth is quantity-based, not a one-status lifecycle.

Conceptually:

```text
available
=
authoritative lot quantity
-
actual sold
-
active remaining reservations
-
finalized quantity adjustments
```

Do not independently maintain editable `available_quantity`.

All inventory mutations go through one domain boundary such as:

```text
InventoryService
- reserve()
- release()
- record_sale()
- split_lot()
- withdraw()
- adjust()
```

Do not scatter `lot.quantity -= ...` throughout the codebase.

Finalized inventory changes are:

- transactional,
- auditable,
- idempotent where required,
- correctable without silently deleting history.

Database constraints should enforce what they safely can.

---

# 12. QuantityReservation

## DESIGN DECISION

Proposals do **not** reserve inventory.

Approval does.

Approval:

```text
current availability recheck
        ↓
transaction / lock
        ↓
create quantity reservation
        ↓
mark exact revision approved
        ↓
commit atomically
```

Track conceptually:

```text
reserved_quantity
consumed_quantity
released_quantity
```

with:

```text
remaining_reserved
=
reserved
-
consumed
-
released
```

Reservation display states such as ACTIVE/PARTIAL/CONSUMED/RELEASED should be **derived**, not independently editable truth.

Cancellation releases only unexecuted reservation.

---

# 13. FINAL-AUDIT CORRECTION — cancellation modeling

## DESIGN DECISION — supersedes the simpler earlier `APPROVED → CANCELLED` picture

Post-approval cancellation is primarily an **operational commitment/reservation action**, not an erasure of the historical decision that the revision was approved.

Why:

- one assignment may be cancelled while others continue,
- only part of an assignment may remain,
- some quantity may already have executed,
- the system must preserve that the manager originally approved the revision.

Therefore:

```text
PlanRevision historical decision:
APPROVED
```

may remain true while:

```text
some/all remaining assignment commitments
→ cancelled/released
```

A UI may derive aggregate labels such as “fully cancelled” or “partially cancelled”, but do not force all cancellation semantics into one mutable `PlanRevision` decision enum.

Already executed sales are never undone by cancellation.

---

# 14. FINAL-AUDIT GAP — off-plan ActualSale vs active reservation

## CONFIRMED facts that create the issue

- ActualSale must be recordable even when off-plan.
- ActualSale fundamentally belongs to ProduceLot.
- SellingAssignment reference is optional.
- inventory subtracts both actual sold and remaining active reservations.
- inventory must never become negative or double-committed.

## DESIGN INVARIANT

`record_sale()` must transactionally preserve:

```text
actual sold
+
remaining active reservations
+
other finalized decrements
<=
authoritative lot quantity
```

## Easy case

If an ActualSale is linked to an approved SellingAssignment:

- consume the corresponding reservation quantity as part of the same inventory transaction.

## Unresolved case

If an **off-plan** ActualSale occurs while the lot has active reservations, the system must **not silently guess which approved reservation to break**.

It must not:

- reject reality merely because a plan exists,
- silently steal quantity from an arbitrary reservation,
- leave sold + reserved greater than authoritative quantity.

### UNKNOWN / MUST VALIDATE BEFORE PILOT

Exact reconciliation workflow, e.g.:

- manager chooses which reservation(s) to release/reconcile,
- explicit deviation/reconciliation operation,
- another auditable resolution mechanism.

This is a newly surfaced final-audit domain gap and must be resolved before final `record_sale()` behavior is deployed.

---

# 15. Storage physical model

## DESIGN DECISION

```text
StorageLocation
      ↓
StorageZone
```

A ProduceLot has one active physical placement at an instant, with placement history.

Physical splitting uses child/sub-lots where required.

Device assignment also has history so old readings remain interpretable after device moves.

---

# 16. IoT / StorageAssessment

## DESIGN DECISION

Pipeline:

```text
ESP32 + DHT22
      ↓
MQTT
      ↓
SensorReading
      ↓
StorageConditionService
      ↓
StorageAssessment
      ↓
optimizer / warnings
```

DHT22 provides ambient:

- temperature,
- relative humidity.

It does **not** directly measure:

- grain moisture,
- grain quality,
- spoilage.

StorageAssessment categories:

```text
HIGH
MEDIUM
LOW
UNKNOWN
```

`UNKNOWN` is mandatory.

Missing/stale/invalid/offline sensor data never silently becomes LOW.

Device health and storage condition are distinct.

Do not claim scientifically validated spoilage prediction from DHT22.

### UNKNOWN before pilot

- validated storage thresholds,
- sampling rate,
- MQTT QoS/topic/auth,
- operational meaning of urgency in real FPO workflow.

Multiple sensors per zone can be deferred.

---

# 17. Forecasting

## DESIGN DECISION

Separate offline training from inference.

```text
OFFLINE
historical data
→ training
→ versioned model artifact

PRODUCTION
model artifact + current input
→ inference
→ PriceForecast
```

Never retrain for every request/replan.

A3T-GCN is a candidate, not a domain dependency.

Optimizer consumes normalized `PriceForecast`, not model internals.

Current forecast contract:

- mandi,
- commodity,
- target date,
- predicted point price,
- generated_at,
- model version,
- availability/status.

Future optional:

- lower/upper bounds,
- uncertainty/confidence metadata.

Weather is optional future input.

---

# 18. Effective optimizer price

## DESIGN DECISION — final-audit future-proofing refinement

Keep domain `PriceForecast` naturally indexed by:

```text
(mandi, commodity, date)
```

but let normalized optimizer economics consume an **effective choice-level price/rate**:

```text
p[l,m,d]
```

For simple V1:

```text
p[l,m,d] = forecast_price[m,d]
```

for all relevant lots.

Later, validated grade/quality economics can change effective lot-level value without changing the core optimizer contract.

Do not fabricate quality adjustments until real requirements are validated.

---

# 19. Routing

## DESIGN DECISION

`RoutingService` is provider-independent.

Current preferred V1 provider may be Google Routes; OSRM/Mapbox/etc. remain replaceable.

Routing produces:

- distance,
- duration,
- status,
- provenance,
- timestamp/freshness metadata.

It does **not** produce FPO-specific rupee transport cost.

Provider failure is not “road blocked”.

Never convert route failure to:

```text
0 km
```

Use explicit unavailable/stale/invalid/error states.

---

# 20. TransportCostService

## DESIGN DECISION

Transport economics are separate from RoutingService.

Conceptually future-capable inputs include:

- origin,
- destination,
- quantity,
- rate/vehicle profile,
- possible grouping,
- tolls,
- quotes.

V1 is **not** fleet management / VRP unless real workflow proves it necessary.

Optimizer receives a frozen deterministic `TransportCostModel` / normalized transport economics; it performs no network calls.

Architecture should support at least future fixed + variable structure:

```text
B*z + c*x
```

without redesign.

Any Big-M/activation bound must be a tight safely derived domain bound, never an arbitrary giant constant.

## UNKNOWN / CORE BLOCKER

- who arranges transport,
- dispatch origin,
- actual pricing method,
- grouping semantics,
- who pays,
- quote validity,
- actual cost recording.

Do not freeze a formula before real FPO validation.

---

# 21. External provider pattern

## DESIGN DECISION

All external provider adapters follow:

```text
request
→ bounded timeout
→ bounded retry where safe
→ validate
→ normalize
→ attach source/status/time
→ domain object
```

No fabricated zero fallbacks.

Provider implementation must not leak into domain contracts.

---

# 22. SellingPlan

## DESIGN DECISION

One V1 SellingPlan concerns:

- one FPO,
- one commodity,
- finite planning window,
- selected eligible ProduceLots,
- one or more candidate mandis,
- planning/allocation policy.

`eligible ≠ selected`.

The manager controls the selected-lot and candidate-mandi search boundary.

Candidate mandi ≠ assigned mandi.

Multiple commodities use separate plans; a future batch may group them.

---

# 23. Canonical identifiers

## DESIGN DECISION

Use canonical internal IDs for:

- FPO,
- Farmer,
- Mandi,
- Commodity,
- StorageLocation,
- etc.

Provider IDs/names map through adapters/mapping records.

Do not join modules using raw strings such as slightly different commodity labels.

Variety/grade remains extensible until validated.

---

# 24. Immutable revision model

## DESIGN DECISION

A SellingPlan has immutable `PlanRevision`s.

Origins may include:

```text
OPTIMIZER
MANAGER_MODIFIED
MANAGER_MANUAL
```

Manager may change decision variables such as:

- mandi,
- date,
- quantity.

Manager may not overwrite derived system facts such as:

- predicted price,
- route distance,
- storage assessment,
- derived transport estimate.

Changing a decision variable creates a **new immutable revision**, recomputes affected derived data, and validates again.

Old revisions are never overwritten.

---

# 25. Revision-specific frozen evaluation basis

## DESIGN DECISION — final-audit refinement

Every evaluated PlanRevision — optimizer, manager-modified, or manual — must be traceable to the exact frozen evidence/policies used to evaluate that revision.

A revision-specific evaluation basis includes as applicable:

- selected lot quantities,
- planning dates,
- candidate mandis,
- forecast IDs/values/versions,
- storage assessments,
- route estimates/providers,
- transport economics/source/version,
- MarketLoadPolicy values/version,
- EconomicPolicy version/parameters,
- FairnessPolicy,
- StoragePolicy,
- numeric normalization policy,
- validation policy versions.

A manager-modified revision must not accidentally reuse stale derived facts from the previous revision.

---

# 26. OptimizationRun scope

## DESIGN DECISION

Optimizer-based plan generation creates an `OptimizationRun`.

Conceptually:

```text
Generate request
→ OptimizationRun
→ build/freeze snapshot
→ input validation
→ optimizer
→ output validation
→ proposed revision if eligible
```

`OptimizationRun` provides:

- async-ready orchestration,
- benchmarkability,
- failure tracking,
- retries,
- reproducibility,
- debugging.

### Important semantic refinement

Do **not** fabricate an `OptimizationRun` for a purely manual revision that never invokes the optimizer merely for symmetry.

Manual/modified revision evaluation still uses frozen evidence + shared validators, but `OptimizationRun` remains about actual optimization execution.

---

# 27. Optimizer boundary

## DESIGN DECISION

Optimizer is computationally isolated:

```text
validated OptimizationInput
        ↓
OptimizationEngine
        ↓
OptimizationResult
```

It:

- performs no network/API calls,
- performs no database writes,
- does not know FastAPI,
- does not know PostgreSQL,
- does not know Google/MQTT/AGMARKNET payloads,
- returns structured solver-neutral result.

This makes it independently unit-testable.

---

# 28. Technical admissibility

Define:

```text
E_tech ⊆ L × M × D
```

for technically admissible lot-mandi-date choices.

A choice may require:

- selected lot,
- candidate mandi,
- allowed date,
- usable forecast,
- usable route,
- usable transport economics.

Storage `UNKNOWN` does not itself make a choice forbidden.

Economic acceptability is separate.

Provider outage is not real-world infeasibility.

Wrong FPO/commodity/malformed units/etc. are input-invalid conditions, not merely masked coordinates.

Prefer sparse model creation over dense `L×M×D` variables where choices are not admissible.

Never silently prune legitimate choices only for speed and then claim global optimality.

---

# 29. Core optimization variables

For admissible `(l,m,d)`:

```text
y[l,m,d] ∈ Z>=0
```

and:

```text
x[l,m,d] = Δ_l * y[l,m,d]
```

where `x` is physical planned quantity.

`x` means:

> proposed quantity from ProduceLot `l` to mandi `m` on date `d`.

It does **not** mean:

- actual sold,
- storage quantity,
- reservation,
- transport capacity,
- whole-lot indicator.

One lot may be split across multiple mandis/days.

---

# 30. Participating quantity

For each selected lot:

```text
0 < Q_plan[l] <= Available_snapshot[l]
```

Partial mode:

```text
S_l = Σ_m,d x[l,m,d]
0 <= S_l <= Q_plan[l]
```

Full mode:

```text
S_l = Q_plan[l]
```

Explicit remainder:

```text
U_l = Q_plan[l] - S_l
```

In partial mode the remainder remains inventory and must be visible.

Incompatible full-mode quantity/increment combinations should be detected before solving where possible.

---

# 31. MarketLoadPolicy

Historical arrivals are an empirical operating reference, not official physical mandi capacity.

Policy modes:

### STRICT

Hard constraint:

```text
Σ_l x[l,m,d] <= C_load[m,d]
```

### ADVISORY

V1 semantics:

- no feasibility effect,
- no optimization penalty,
- post-solve context/warning only.

### DISABLED

No effect.

Strict missing required values never become `0` or infinity.

Future `PREFER_WITHIN_LIMIT` is possible but not V1.

Freeze provenance:

- value,
- unit,
- source,
- calculation version,
- timestamp,
- enforcement mode.

## UNKNOWN

Exact calibration/window and final FPO enforcement choice.

If STRICT is used in production, the empirical model requires stronger validation than advisory use.

---

# 32. EconomicPolicy

## DESIGN BOUNDARY

`EconomicPolicy` answers:

> Is this candidate/plan economically acceptable enough to be considered?

It is versioned and deterministic.

It may eventually contribute:

- choice-level requirements,
- farmer-level requirements,
- whole-plan requirements.

It is not merely “minimum price” unless validated.

It must preserve exclusion reasons/provenance.

`EconomicPolicy` defines the allowed region; expected net return later ranks allowed solutions.

Negative NER is not automatically forbidden unless policy says so.

## UNKNOWN / CORE BLOCKER

Exact FPO economic acceptability rule.

Manager override semantics are also unknown.

---

# 33. Expected net selling return

Use the term:

> **expected net selling return**

not “profit”.

Conceptually:

```text
gross
=
Σ p[l,m,d] * x[l,m,d]
```

and:

```text
expected_net_return
=
gross
-
modeled_transport_cost
```

Transport aggregation may evolve.

Estimated transport cost remains separate from actual expense.

---

# 34. Allocation fairness

## Current V1 DESIGN DECISION

Fairness is per farmer, not per lot.

A farmer with three lots does not receive three fairness votes.

For participating farmer `f`:

```text
P_f = Σ Q_plan[l] for lots owned by f
S_f = Σ x[l,m,d] for lots owned by f
r_f = S_f / P_f
```

### Final-audit clarification

The fairness set includes only participating farmers with:

```text
P_f > 0
```

No zero denominator.

Current fairness semantics:

> true lexicographic max-min (`leximin`) farmer fulfilment.

Sort fulfilment ratios:

```text
r_(1) <= r_(2) <= ... <= r_(|F|)
```

and lexicographically maximize the full sorted vector.

This is stronger than only maximizing the single worst ratio.

Outcome/net-return fairness remains report-only initially.

Historical fairness is future-ready, not V1.

### MUST VALIDATE BEFORE PILOT

The real FPO's fairness practice should be tested against this policy.  
Do not hard-code leximin across the entire application; keep it a versioned `FairnessPolicy`.

---

# 35. Leximin reference semantics

Define:

```text
L_k = sum of the k smallest farmer fulfilment ratios
```

True leximin can be defined by sequentially maximizing:

```text
L_1, L_2, ..., L_|F|
```

while preserving every previous optimum.

Do not identify/freeze “worst named farmers” merely from one arbitrary solver solution.

That can be wrong when multiple equivalent bottleneck assignments exist.

---

# 36. MILP leximin representation

## ENGINEERING FORMULATION

For each fairness prefix `k`, introduce:

```text
0 <= θ_k <= 1
u[f,k] >= 0
u[f,k] >= θ_k - r_f
```

Then:

```text
k*θ_k - Σ_f u[f,k]
```

represents the sum of the `k` smallest fulfilment ratios when maximized.

This gives an exact mathematical MILP representation of the chosen semantics.

Potential cost:

```text
O(|F|^2)
```

fairness auxiliaries in the straightforward full prefix formulation.

A more efficient mathematically equivalent progressive bottleneck implementation may be benchmarked later.

Solver floating arithmetic/tolerances are not identical to symbolic exact rational arithmetic; domain fairness truth is always recomputed from exact numerator/denominator quantities.

---

# 37. Storage objective

Current design after higher priorities:

For `HIGH` lots:

1. maximize total HIGH quantity scheduled;
2. preserve it;
3. lexicographically maximize HIGH quantity on earliest date, then next date, etc.

Then repeat for `MEDIUM`.

`LOW` has no extra early-sale pressure.

`UNKNOWN` gets no fabricated rank.

Use only the current frozen StorageAssessment; do not invent a deterioration trajectory.

### Full-mode simplification

When every participating lot must be fully allocated:

- total allocation is already fixed,
- all farmer fulfilment ratios are 1,
- total HIGH scheduled is fixed,
- total MEDIUM scheduled is fixed.

Only HIGH/MEDIUM **timing** can still affect assignment before economics.

### IMPORTANT CONDITIONALITY

Storage priority is currently before NER, but the handoff also requires that storage urgency must not blindly create enormous economic sacrifice.

Therefore this hierarchy is production-safe only once a validated `EconomicPolicy` supplies the intended economic guardrail, or the hierarchy is revisited.

### MUST VALIDATE BEFORE PILOT

Real FPO storage-vs-price decision practice.

---

# 38. Objective hierarchy

Current preferred hierarchy:

```text
0. hard technical/domain validity
1. economic acceptability
2. maximize useful allocation (partial mode)
3. farmer allocation fairness
4. storage priority
5. maximize expected net selling return
6. deterministic technical tie-break
```

Do not mix rupees, fairness, and urgency using arbitrary mega-weights such as:

```text
0.6 profit + 0.25 storage + 0.15 fairness
```

Use lexicographic priority semantics.

---

# 39. Lexicographic objective preservation

Reference semantics:

```text
optimize J1
→ preserve J1*
→ optimize J2
→ preserve J2*
→ ...
```

Partial mode:

1. maximize total allocation `A`;
2. preserve `A*`;
3. complete leximin fairness;
4. HIGH total + HIGH date vector;
5. MEDIUM total + MEDIUM date vector;
6. NER;
7. deterministic tie-break.

Full mode skips redundant allocation/fairness stages as described above.

The production implementation may use native hierarchical objectives, incremental solving, or equivalent encodings, but the domain meaning is solver-independent.

---

# 40. Deterministic tie-break

After all business objectives are tied, choose a deterministic canonical representative.

Current conceptual coordinate ordering:

```text
lot canonical ID
→ date
→ mandi canonical ID
```

Exact efficient implementation is not frozen.

Do not implement one solver call per variable or giant exponential objective coefficients without benchmark evidence.

Tie-break is technical, not business preference.

---

# 41. OptimizationInput

## DESIGN DECISION

Frozen Snapshot is richer than mathematical OptimizationInput.

OptimizationInput contains normalized solver-neutral items such as:

- ordered `F, L, M, D`,
- `E_tech`,
- `owner(l)`,
- `Q_plan[l]`,
- planning increment `Δ_l`,
- effective price `p[l,m,d]`,
- StorageAssessment category `u_l`,
- TransportCostModel,
- MarketLoadPolicy math data,
- EconomicPolicy resolved math specification,
- AllocationPolicy,
- FairnessPolicy/version,
- StoragePolicy/version,
- deterministic tie-break definition/version,
- normalization policy/version.

It does **not** contain:

- ORM objects,
- Google responses,
- MQTT messages,
- raw model internals,
- DB sessions,
- network clients.

Solver execution parameters such as timeout/seed/threads/tolerances live in `OptimizationExecutionConfig`, not domain OptimizationInput.

---

# 42. OptimizationResult

## DESIGN DECISION

OptimizationResult is:

- not a proposed revision,
- not an approved plan,
- not a reservation,
- not an ActualSale.

It contains structured solution/proof information.

Conceptually separate:

### solution availability
- candidate exists / none.

### feasibility evidence
- proven feasible candidate,
- proven infeasible where the solver can prove it,
- unknown/not proven.

### allocation outcome
- FULL,
- PARTIAL,
- NONE / no useful allocation.

### optimality evidence
- PROVEN_OPTIMAL,
- FEASIBLE_NOT_FULLY_PROVEN.

### execution termination
- completed,
- time limit,
- interrupted,
- technical failure,
- etc.

Exact enums remain to be finalized in code-contract work.

Sparse assignments return physical exact quantities:

```text
(lot_id, mandi_id, date, quantity)
```

not internal `y` variables.

Include:

- per-lot participating/scheduled/unallocated,
- farmer fairness numerator/denominator,
- storage outcomes,
- gross,
- modeled transport,
- expected net return,
- solver-neutral diagnostics,
- objective hierarchy prefix certified if optimization stops early.

A zero-allocation result may be mathematically feasible in partial mode but is not necessarily a useful proposal.

Whether feasible-but-not-fully-optimized results are proposal-eligible remains UNKNOWN/product policy.

---

# 43. Solver strategy

## ENGINEERING LEAN — not frozen production solver

Architecture:

```text
OptimizationEngine
    ↓
replaceable implementation
```

Current first formulation to prototype:

> MILP.

Why:

- integer allocation quantities,
- linear capacity/accounting,
- linear/fixed-charge transport support,
- linear economics,
- natural continuous fairness auxiliaries.

Benchmark against:

- SCIP — open-source MILP deployment candidate,
- Gurobi — high-performance/reference MILP benchmark subject to licensing,
- HiGHS — additional open-source MILP baseline,
- CP-SAT — exact discrete alternative,
- heuristic — benchmark/fallback/research comparison.

Do not make the domain depend on any solver library.

A modeling layer such as MathOpt may be useful inside the optimization module, but it is not the architecture.

### Selection gates

1. Correctness.
2. Failure/status semantics.
3. Deployment/licensing acceptability.
4. Performance at validated FPO scale.
5. Then operational trade-offs.

Do not use a weighted “solver score”.

---

# 44. Solver verification

## DESIGN DECISION

Create an independent tiny brute-force/oracle suite for small scenarios.

Differentially compare exact solvers using the same frozen business semantics.

For proven-optimal cases, compare the complete business objective tuple, not just NER.

Always pass returned plans through independent PlanValidator.

Benchmark:

- model build time,
- solve time,
- validation time,
- peak memory,
- `|L|`, `|F|`, `|M|`, `|D|`, `|E_tech|`,
- admissibility density,
- variable/constraint counts,
- termination,
- proof/gap where meaningful,
- number of objective stages,
- repeatability.

---

# 45. Scalability

Primary allocation width:

```text
N_allocation = |E_tech|
```

rather than necessarily dense:

```text
|L| * |M| * |D|
```

Fairness can introduce approximately:

```text
O(|F|^2)
```

auxiliaries in the straightforward exact prefix formulation.

Transport may add binary/grouping variables depending on the final model.

Objective depth also grows with:

```text
|F| + |D|
```

because of leximin fairness and date-priority stages.

Do not invent production size caps until real FPO sizes and benchmarks exist.

Do not decompose by farmer/mandi/time and claim equivalence unless mathematically proven; such decomposition would often change the problem.

---

# 46. Numerical architecture

## DESIGN DECISION

Three layers:

```text
exact domain values
        ↓
exact/lossless optimization normalization
        ↓
solver numerical execution
        ↓
exact domain reconstruction
        ↓
PlanValidator
```

Rules:

- solver quantity normalization must be lossless;
- no hidden quantity rounding;
- raw solver float never becomes reservation/inventory quantity;
- reconstruct exact integer decisions before domain use;
- fairness business truth uses exact `S_f/P_f`, not solver `r_f`;
- no universal `EPS` across inventory/money/fairness/solver numerics;
- solver tolerances belong in `OptimizationExecutionConfig`;
- record solver/version/config for reproducibility;
- numerical preparation failure is technical failure, never business infeasibility;
- add numeric preflight checks for NaN/Infinity/scales/coefficient magnitude/Big-M/overflow.

---

# 47. Validation layers

## DESIGN DECISION

Three conceptually distinct validation layers:

### 1. OptimizationInputValidator

Checks prepared input, e.g.:

- FPO consistency,
- commodity consistency,
- exact positive quantities,
- availability snapshot,
- dates,
- candidate mandis,
- forecast availability,
- transport availability,
- units/scales,
- policy inputs,
- full-mode increment compatibility.

### 2. PlanValidator

Shared deterministic core for:

- optimizer proposal,
- manager-modified plan,
- manager-manual plan.

Checks:

- correct FPO/commodity,
- only allowed lots,
- positive/exact quantities,
- scheduling increment,
- per-lot totals/remainders,
- full vs partial policy,
- no over-allocation,
- allowed dates/mandis,
- technical admissibility,
- STRICT MarketLoadPolicy,
- EconomicPolicy hard requirements,
- recomputed economic arithmetic.

It does not prove solver optimality.

It does not secretly call external providers.

Manager may choose a hard-valid plan that is less fair/lower NER/later storage than the optimizer optimum unless policy explicitly makes those hard requirements.

### 3. Approval-time validation

Uses current authoritative commit context:

- current inventory,
- current reservation conflicts,
- revision state,
- authorization,
- execution date validity,
- freshness assessment/policy.

Approval performs final inventory lock/recheck atomically with reservation + approval commit.

---

# 48. Proposal freshness

## DESIGN DECISION

Distinguish:

```text
historical revision validity
≠
current commit validity
≠
freshness
```

Old revision keeps the exact frozen evidence under which it was evaluated.

Later external changes never silently rewrite it.

Freshness may consider:

- forecast,
- storage,
- route,
- transport rate/quote,
- market-load policy,
- economic policy,
- other relevant versions.

Freshness detection and approval action are separate.

Conceptual observations may include:

- unchanged/fresh,
- conditions changed,
- explicitly expired,
- newer version exists,
- unavailable,
- not verified/unknown.

Do not use one universal TTL for every evidence type.

If a material non-blocking freshness issue is detected at approval, manager should be explicitly informed before commitment.

If active policy makes it blocking, refresh/revision/replan is required.

Exact thresholds and warning/blocking rules remain UNKNOWN.

Do not auto-replan on every small market/sensor change in V1.

---

# 49. Approval flow

Conceptually:

```text
manager requests approval
      ↓
resolve exact immutable revision
      ↓
authorization + idempotency
      ↓
prepare ApprovalContext
      ↓
hard commit checks
      ↓
freshness policy
      ↓
if allowed, short DB transaction
      ↓
lock/recheck inventory
      ↓
create reservations
      ↓
mark revision approved
      ↓
commit
```

Do not hold DB inventory locks while waiting on network providers.

Freshness checks happen before the critical inventory transaction.

---

# 50. PlanRevision states

## DESIGN DECISION

Before approval:

```text
PROPOSED
   ├──→ APPROVED
   ├──→ REJECTED
   └──→ SUPERSEDED
```

`SUPERSEDED` means replaced by a newer revision; it is not the same as rejection.

A superseded/rejected revision is never reopened and approved later.  
Create a new revision instead.

An approved revision is never edited back into proposal state.

### Post-approval refinement

Do not overload PlanRevision decision state with every partial cancellation/execution condition.

Historical approval remains preserved while operational commitment quantities may later be consumed/released/cancelled.

---

# 51. Revision vs replan

## DESIGN DECISION

### Revision

Same planning cycle, pre-commit decision-making:

```text
proposal
→ manager changes decision variables
→ new immutable revision
```

### Replan

After commitment/reality/current inventory changes materially:

```text
approved plan
→ execution/deviation/cancellation/remaining quantity
→ current authoritative inventory
→ current facts/policies
→ new OptimizationRun
→ new SellingPlan
```

### Final-audit clarification

After a revision has been approved, business changes should not normally be modeled by editing/adding another revision under the same committed plan.

Use:

- cancellation/release of remaining commitment as necessary,
- then a new current-data planning cycle.

Old approved/executed history remains unchanged.

---

# 52. Communication

## DESIGN DECISION

Approved plan is the source of operational instructions.

```text
Approved assignment
→ CommunicationService
→ Web / WhatsApp / future channel
```

WhatsApp must not independently rerun optimizer.

Communication state is separate:

```text
NOT_SENT
SENT
DELIVERED
FAILED
```

exact enum later.

Communication failure does not silently undo approval/reservation.

Future DeliveryRecord may capture channel/status/timestamps.

---

# 53. Planned vs actual execution

## DESIGN DECISION

Never overwrite recommendation with reality.

Preserve both:

```text
PLANNED
mandi/date/quantity/predicted price
```

and:

```text
ACTUAL
mandi/date/quantity/actual price
```

This supports:

- forecast evaluation,
- recommendation adherence,
- manager override analysis,
- optimizer evaluation.

---

# 54. ActualSale

## DESIGN DECISION

ActualSale fundamentally belongs to ProduceLot.

SellingAssignment reference is optional.

Conceptually:

```text
ActualSale
- ProduceLot
- SellingAssignment? optional
- actual mandi
- actual date
- actual quantity
- actual price
- recorded_at
```

One SellingAssignment may have 0..many ActualSales.

A 40q assignment may execute as 20q + 15q + 5q.

Off-plan ActualSale must still be recordable.

Potential future `ExecutionAttempt` for failed trips is explicitly deferred unless real workflow requires it.

---

# 55. Execution state

Execution is derived from actual quantities, not manually toggled.

Conceptually for an assignment:

```text
0 sold
→ NOT_STARTED

0 < sold < planned/committed target
→ PARTIAL

sold reaches planned/committed target
→ COMPLETED
```

Reality may deviate from recommendation; deviation analytics remain separate.

If actual sale exceeds an assignment's planned/reserved quantity, the extra quantity becomes an inventory/reconciliation deviation and must still preserve lot-level inventory invariants.

Exact deviation reconciliation workflow remains to be validated.

---

# 56. Orthogonal state model

Avoid giant statuses like:

```text
APPROVED_AND_WHATSAPPED_AND_PARTIALLY_SOLD_AND_STALE
```

Keep separate:

```text
OptimizationRun lifecycle
Optimization outcome
PlanRevision decision
Freshness
Communication
Execution
Reservation
```

Example:

```text
OptimizationRun: FINISHED / proposal created
Revision decision: APPROVED
Freshness: CONDITIONS_CHANGED
Communication: DELIVERED
Execution: PARTIAL
Remaining reservation: 15q
```

This is intentional and makes debugging much easier.

---

# 57. OptimizationRun lifecycle vs outcome

## DESIGN DECISION

Do not mix:

```text
RUNNING
```

with:

```text
INFEASIBLE
```

or:

```text
PARTIAL
```

in one axis.

Conceptually:

### lifecycle

```text
PREPARING
READY
RUNNING
FINISHED
CANCELLED
```

exact names may change.

### outcome

Concepts such as:

- proposal created,
- input invalid,
- proven infeasible,
- no useful allocation,
- solver failed,
- output validation failed.

Solution availability, allocation outcome, proof quality, and termination remain separately reportable.

A manager rejecting the proposal never retroactively changes the OptimizationRun from success to failure.

Communication failure also does not change run success.

---

# 58. Reliability philosophy

## DESIGN DECISION

Unknown/unavailable is first-class.

Examples:

```text
forecast unavailable
→ unavailable, not ₹0

route unavailable
→ unavailable, not 0km

transport rate missing
→ unavailable, not ₹0

sensor stale
→ UNKNOWN, not LOW

optimizer proven infeasible
→ infeasible, not system error

solver crash
→ technical failure, not “no feasible plan”
```

Fail visibly rather than return believable nonsense.

---

# 59. Idempotency and concurrency

Required where duplicates/races are plausible:

- MQTT readings,
- plan generation,
- approval,
- cancellation/release,
- ActualSale recording,
- communication webhooks.

Use:

- idempotency keys/event IDs where appropriate,
- DB constraints,
- transactions/locking,
- tenant checks.

Double-clicking approve must not reserve twice.

Critical inventory and approval operations are transactional.

---

# 60. Time

## DESIGN DECISION

Technical timestamps stored in UTC.

FPO has explicit local timezone for business dates/times.

Do not use naive timestamps.

Same-day sale/approval cutoff rules remain UNKNOWN until workflow validation.

---

# 61. Audit and feedback

Keep separate:

### Audit
answers:

- WHO,
- WHAT,
- WHEN.

### DecisionFeedback
answers:

- WHY.

Capture structured reasons for meaningful:

- rejection,
- approved modification,
- cancellation/release,
- execution deviation.

Use reason code + optional note.

Feedback does not automatically retrain forecasting/storage/optimizer models.

Correct loop:

```text
feedback
→ analytics
→ human investigation
→ new validated version
```

No event-sourcing platform required for V1.

---

# 62. Testing strategy

## DESIGN DECISION

### Unit
- exact inventory arithmetic,
- reservation accounting,
- transport-cost logic,
- forecast normalization,
- storage assessment,
- fairness,
- optimizer,
- validators.

### Provider contract tests
- routing adapter,
- market adapter,
- MQTT,
- WhatsApp.

### Integration
- FastAPI ↔ PostgreSQL,
- approval transaction,
- concurrent approval,
- reservation consume/release,
- off-plan sale reconciliation once policy is resolved.

### End-to-end
Example:

```text
create farmer
→ register lot
→ storage placement
→ generate proposal
→ modify if desired
→ approve
→ communicate
→ partial ActualSale
→ reservation consumption/release
→ replan current remainder
→ final sale
```

### Property/randomized
Random reserve/release/sell/split sequences must never:

- make quantity negative,
- exceed authoritative total,
- create double reservation,
- violate FPO isolation.

### Additional final-audit tests
- frozen snapshot replay,
- differential solver tests,
- brute-force oracle for tiny optimizer cases,
- failure injection,
- revision lineage,
- stale/freshness cases,
- audit reconstruction.

---

# 63. Real-FPO unknowns — priority classification

## A. CORE BUSINESS CONTRACT BLOCKERS

Must be validated before the affected production behavior is frozen:

1. Where produce physically remains before sale.
2. Farmer→mandi vs warehouse/collection-center→mandi execution path.
3. Who arranges transport.
4. Actual transport pricing method.
5. Exact economic acceptability rule.
6. Minimum optimizer scheduling increment.
7. Whether quality/variety materially affects eligibility/effective price.
8. MarketLoadPolicy mode/calibration if used as STRICT.
9. **Off-plan ActualSale vs active-reservation reconciliation workflow** (new final-audit gap).

These do not prevent stable foundation code, but they block final affected policy/business operations.

---

# 64. Real-FPO unknowns — must validate before pilot

1. Produce registration workflow.
2. Who verifies quantity/quality.
3. Exact staff/manager authority.
4. Whether farmer must explicitly accept before execution.
5. Who records ActualSale.
6. Whether actual transport expense is reliably recorded.
7. Scientific/operational storage thresholds.
8. IoT deployment details.
9. Current-run fairness policy acceptance by managers.
10. Storage-vs-economic-priority behavior.
11. Approval/freshness warning/block rules.
12. Same-day operational cutoffs.

---

# 65. Safe to defer

Do not block V1 foundation on:

- historical fairness policy,
- multiple sensors per zone,
- advanced WhatsApp self-service,
- payment/accounting,
- weather,
- fleet/VRP,
- full logistics optimization,
- failed `ExecutionAttempt` entity,
- advanced quality models,
- automatic model retraining from feedback.

Architecture remains future-ready.

---

# 66. FPO interview priority

Ask in operational vocabulary, not solver vocabulary.

High-value questions:

1. When farmers have produce ready, where is it physically kept?
2. Who decides today where/when/how much to sell?
3. If several farmers sell tomorrow, who arranges transport?
4. How exactly does transporter charge?
5. When would you refuse a mandi even if its predicted price is highest?
6. What is the smallest quantity you realistically schedule?
7. Does variety/quality change mandi choice or expected selling price?
8. Do you enforce any practical daily mandi/load limit?
9. Can a farmer refuse/need to accept the manager's decision?
10. Who records final actual quantity/price?
11. What happens if a farmer sells elsewhere after quantity was already committed in an FPO plan?

Do not lead the interview with terms like MILP, CP-SAT, max-min, MQTT QoS, etc.

If real evidence conflicts with this handoff, mark **CONFLICT** before changing design.

---

# 67. Revised/rejected ideas — never revive silently

### REVISED
“Selected quantity must always be fully allocated.”

Current:

- both full and partial allocation policies exist;
- partial remainder remains explicit inventory.

### REVISED
“Approved plan can only be cancelled before execution.”

Current:

- only remaining unexecuted commitment is released;
- executed ActualSales remain reality.

### REFINED
“ActualSale belongs to SellingAssignment.”

Current:

- ActualSale belongs fundamentally to ProduceLot;
- assignment reference optional.

### REFINED
“Rule-based spoilage prediction from DHT22.”

Current:

- conservative environmental StorageAssessment;
- no unsupported direct spoilage claim.

### REFINED
“OSRM is routing.”

Current:

- provider-independent RoutingService;
- provider replaceable.

### REFINED
“Historical arrival number is mandi physical capacity.”

Current:

- empirical versioned MarketLoadPolicy/reference;
- STRICT/ADVISORY/DISABLED semantics configurable.

### REFINED
“PlanRevision APPROVED → CANCELLED is the universal cancellation model.”

Current:

- approval remains historical fact;
- post-approval cancellation/release is quantity-aware operational commitment state;
- aggregate cancellation may be derived.

### REFINED
“p[m,d] is permanently sufficient.”

Current:

- domain PriceForecast remains mandi/commodity/date;
- optimizer accepts future-ready effective `p[l,m,d]`;
- V1 can map it directly to mandi/day forecast.

### CORRECTED
“MILP is already the frozen production solver.”

Current:

- MILP is leading formulation to prototype;
- production solver must pass benchmark gates.

---

# 68. Things that must NOT be invented in implementation

Do not silently decide:

- transport formula,
- economic threshold,
- minimum allocation increment,
- produce origin,
- farmer consent requirement,
- quality irrelevance,
- STRICT market-load use,
- freshness TTLs,
- solver winner,
- solver timeout/gap policy,
- arbitrary global EPS,
- generic “everything settings” key-value blob,
- post-approval off-plan reservation reconciliation.

Unknowns remain explicit until validated.

---

# 69. Safe implementation order

Architecture is now sufficiently mature to start stable foundation work **while unresolved policy blockers are being validated**.

## Phase 0 — protect the source of truth

Before coding:

1. keep this handoff in the repository/docs;
2. list unresolved blockers;
3. define very small implementation tasks;
4. no AI coding tool may invent entities/workflows/statuses/policies.

For each coding task state:

- scope,
- contract,
- explicit exclusions,
- tests,
- completion report.

---

## Phase 1 — project skeleton and core value objects

Implement only stable concerns:

- modular FastAPI structure,
- PostgreSQL migrations,
- FPO tenant identity,
- canonical IDs,
- UTC/local-time handling,
- exact Quantity,
- exact Money/Rate,
- basic audit/request IDs,
- authorization boundary skeleton.

Tests first.

---

## Phase 2 — Farmer / ProduceLot / inventory core

Implement:

- FarmerMember,
- ProduceLot,
- exact quantity invariants,
- InventoryService boundary,
- QuantityReservation,
- reservation arithmetic,
- transactions/idempotency,
- split/withdraw/adjust skeleton only where contracts are frozen.

Do not finalize off-plan sale reconciliation until the newly identified workflow question is resolved.

Property-test inventory heavily.

---

## Phase 3 — storage and provider-neutral derived-data contracts

Implement stable contracts:

- StorageLocation/Zone/Placement,
- DeviceAssignment history,
- SensorReading,
- StorageAssessment with UNKNOWN,
- PriceForecast,
- RouteEstimate,
- provider abstractions.

Do not overclaim storage science.

---

## Phase 4 — planning/revision/evaluation contracts

Implement:

- SellingPlan,
- immutable PlanRevision,
- SellingAssignment,
- revision origin,
- revision lineage/SUPERSEDED semantics,
- frozen evaluation basis,
- OptimizationRun,
- OptimizationInput/Result,
- validators.

Do not yet hard-code unresolved policies.

---

## Phase 5 — policy interfaces

Implement typed versioned boundaries:

- AllocationPolicy,
- EconomicPolicy interface,
- MarketLoadPolicy,
- FairnessPolicy,
- StoragePolicy,
- ApprovalFreshnessPolicy,
- TransportCostModel interface,
- NumericNormalizationPolicy.

For unresolved ones, provide test fixtures/dummy explicit development implementations only if clearly marked non-production and never presented as validated FPO policy.

Avoid one generic “settings” dictionary.

---

## Phase 6 — optimizer reference and verification

Implement:

- tiny brute-force oracle,
- exact math contract tests,
- sparse `E_tech`,
- leximin reference tests,
- MILP formulation prototype,
- differential solver benchmark harness.

Do not choose production solver from intuition.

---

## Phase 7 — resolve FPO blockers before production optimizer freeze

Must resolve:

- origin/custody,
- transport arranger/pricing/grouping,
- economic policy,
- allocation increment,
- quality relevance,
- MarketLoadPolicy operational mode,
- off-plan sale/reservation reconciliation.

Then freeze actual V1 policy implementations.

---

## Phase 8 — approval/freshness

Implement:

- ApprovalContext,
- current inventory recheck,
- freshness assessment boundary,
- atomic approval + reservation,
- manager warning/block semantics after policy validation,
- audit evidence.

No provider calls inside PlanValidator.

---

## Phase 9 — execution and communication

Implement:

- ActualSale,
- assignment-linked sale reservation consumption,
- cancellation/release,
- approved-instruction communication,
- communication state,
- execution derivation,
- feedback/audit.

Only implement off-plan sale with active reservations after reconciliation semantics are resolved.

---

## Phase 10 — UI / end-to-end pilot

Build manager flows only after contracts underneath them are stable:

- create/select planning problem,
- inspect proposal,
- modify,
- approve,
- freshness warning,
- execution recording,
- replan,
- audit/debug views.

Validate with actual FPO workflow before calling the system production-ready.

---

# 70. Current readiness

## Conceptual architecture

Approximately **complete**.

No further broad architecture brainstorming is required unless:

- real FPO evidence conflicts with an assumption,
- implementation reveals a genuine invariant contradiction,
- benchmarks invalidate the current solver formulation.

## What remains

The remaining major uncertainty is **domain validation**, not architecture invention.

Core unresolved production blockers are explicitly listed above.

Stable foundation implementation may now begin in small steps.

---

# 71. Final mental model

```text
FARMER / PRODUCE REALITY
        ↓
authoritative ProduceLot + placement + inventory
        ↓
normalized derived information
forecast / storage / route / transport
        ↓
revision-specific frozen evaluation basis
        ↓
OptimizationInputValidator
        ↓
replaceable OptimizationEngine
        ↓
OptimizationResult
        ↓
PlanValidator
        ↓
immutable proposal revision
        ↓
manager modify / reject / approve
        ↓
approval-time current checks + freshness
        ↓
atomic reservation
        ↓
approved assignments
        ↓
communication
        ↓
REAL WORLD ActualSale
        ↓
transactional reservation/inventory reconciliation
        ↓
remaining current inventory
        ↓
new planning cycle if needed
```

The system should be able to answer, for any important result:

```text
What facts were used?
Which versions/policies were used?
What did the optimizer return?
What was proven?
What did validation check?
What did the manager decide?
What was reserved?
What was communicated?
What actually happened?
What changed afterward?
```

If it cannot answer those questions, implementation has violated the architecture.

---

# 72. Next-chat instruction

Do **not** restart architecture from scratch.

Next work should be one of:

1. real-FPO validation/interview preparation and recording answers; or
2. implementation Phase 1 using this handoff.

When coding starts, do one very small task at a time, test it, report completion, then continue.

