# Architecture → Implementation Constraint Catalogue

## Phase 1 scope

This catalogue translates the complete [KisanNiti consolidated architecture handoff](../architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md) into technology-independent obligations: what every conforming implementation must guarantee, not which mechanism must provide the guarantee.

The architecture remains authoritative. This document does not freeze the overall Implementation Blueprint, resolve an `UNKNOWN`, select a technology/version, or authorize product coding. The architecture's older proposed implementation order is treated as process guidance; the current verified blueprint process governs the present task.

## Catalogue summary

| Derived constraint family | Count |
|---|---:|
| Human agency and information provenance | 2 |
| Module and dependency boundaries | 2 |
| Tenancy and authorization | 2 |
| Domain identity and physical model | 4 |
| Numeric semantics | 3 |
| Inventory and reservations | 5 |
| Storage and IoT | 2 |
| Forecasting | 2 |
| Providers, routing, and transport | 3 |
| Planning and revision history | 5 |
| Policies, economics, and objectives | 7 |
| Optimization contracts | 9 |
| Validation and approval | 3 |
| Execution and communication | 4 |
| Reliability, state, time, and audit | 5 |
| Verification | 1 |
| UNKNOWN validation gates | 2 |
| **Total** | **61** |

Classification is non-exclusive where a stable `DESIGN DECISION` surrounds an unresolved `UNKNOWN`. The catalogue contains one `CONFIRMED` constraint, 58 constraints carrying `DESIGN DECISION`, one carrying `ENGINEERING LEAN`, and 20 carrying an `UNKNOWN relationship`.

## Human agency and information provenance

### IC-AGENCY-001 — Human-controlled decision support

- **Constraint family:** Human agency and information provenance
- **Architecture classification:** `CONFIRMED`
- **Source:** §2, “Product definition”; §52, “Communication”
- **Architecture statement / rationale:** KisanNiti proposes coordinated selling plans for FPO managers; it does not autonomously sell, and operational instructions derive from an approved plan.
- **Technology-independent implementation obligation:** Preserve the manager as the decision authority and prevent a system proposal from becoming a sale or operational instruction without the required human decision.
- **Required invariant:** Proposal, approval, communication, and real sale remain distinct events; only approved assignments may source operational instructions.
- **Failure semantics:** Proposal generation or communication must never be reported as autonomous sale execution.
- **Audit / debugging implications:** Retain who made the human decision, which proposal/revision was decided, and which approved assignment produced an instruction.
- **Testing implications:** Verify that proposals cannot directly create ActualSales or operational instructions and that communication resolves an approved assignment.
- **Security / tenant-isolation implications:** Approval and instruction actions require authoritative authorization within the owning FPO.
- **Related UNKNOWN / blocker references:** `PILOT-003`, `PILOT-004`, `PILOT-005`.
- **Explicit non-decisions:** Approval role, farmer-acceptance behavior, communication channel, and API/UI mechanism are not selected.
- **Notes:** None.

## Policies, economics, and objectives

### IC-MARKETLOAD-001 — Treat empirical market load through an explicit versioned policy

- **Constraint family:** Policies, economics, and objectives
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §31, “MarketLoadPolicy”; §47, “Validation layers”; §67, “Revised/rejected ideas”
- **Architecture statement / rationale:** Historical arrivals are an empirical operating reference, not physical mandi capacity; `STRICT`, `ADVISORY`, and `DISABLED` have distinct semantics and frozen provenance.
- **Technology-independent implementation obligation:** Evaluate market-load evidence through an explicit versioned policy whose enforcement mode and required data semantics are unambiguous.
- **Required invariant:** `ADVISORY` has no feasibility or objective effect; `DISABLED` has no effect; `STRICT` applies only with usable required values and never substitutes zero/infinity for missing data.
- **Failure semantics:** Missing strict data makes the required policy input unavailable/invalid, not an arbitrary capacity; advisory evidence yields context/warning only.
- **Audit / debugging implications:** Record value, unit, source, calculation version, timestamp, mode, and any warning/exclusion.
- **Testing implications:** Verify each mode, missing/stale values, provenance replay, strict validation, and absence of advisory optimization effects.
- **Security / tenant-isolation implications:** FPO-specific policy selection/configuration remains tenant-scoped.
- **Related UNKNOWN / blocker references:** `CORE-008`.
- **Explicit non-decisions:** Production mode, calibration/window, data source, persistence, and future prefer-within-limit behavior are not selected.
- **Notes:** Protects `RR-006`.

### IC-ECONOMIC-001 — Enforce economic admissibility through a versioned policy boundary

- **Constraint family:** Policies, economics, and objectives
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §32, “EconomicPolicy”; §37, “Storage objective”; §38, “Objective hierarchy”; §47, “Validation layers”
- **Architecture statement / rationale:** EconomicPolicy deterministically defines the allowed region and preserves exclusion provenance; the actual FPO acceptability and override rules are unknown.
- **Technology-independent implementation obligation:** Keep economic admissibility as an explicit versioned policy boundary capable of choice-, farmer-, and plan-level requirements without assuming a threshold.
- **Required invariant:** Only requirements in the frozen policy restrict admissibility; expected net return ranks allowed solutions and does not independently forbid negative values.
- **Failure semantics:** Missing/unresolved production policy prevents affected production behavior from being frozen; it is not replaced by a minimum-price or non-negative-return guess.
- **Audit / debugging implications:** Record policy version/parameters, evaluated scope, evidence, exclusion reasons, and any authorized override once defined.
- **Testing implications:** Verify deterministic evaluation, exclusion provenance, scope levels, and no implicit negative-return/minimum-price rule.
- **Security / tenant-isolation implications:** Policy configuration and override authority must remain FPO-scoped and authorized.
- **Related UNKNOWN / blocker references:** `CORE-005`, `PILOT-003`, `PILOT-010`.
- **Explicit non-decisions:** Threshold/rule, override semantics, policy representation, parameter values, and implementation mechanism are not selected.
- **Notes:** Required economic guardrail remains unresolved.

### IC-ECONOMIC-002 — Preserve expected-net-return semantics and estimated/actual separation

- **Constraint family:** Policies, economics, and objectives
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §3, “Fundamental architecture rule”; §33, “Expected net selling return”; §42, “OptimizationResult”; §53, “Planned vs actual execution”
- **Architecture statement / rationale:** Expected net selling return is normalized gross less modeled transport cost, is not called profit, and remains distinct from actual price/expense.
- **Technology-independent implementation obligation:** Compute, report, and validate expected net selling return from exact normalized planning values while preserving every modeled component separately from actual outcomes.
- **Required invariant:** Reported expected net return equals its frozen gross and modeled-transport components under their recorded version/units; no actual expense overwrites an estimate.
- **Failure semantics:** Unavailable required price/transport evidence prevents a valid computed value; it never becomes zero.
- **Audit / debugging implications:** Retain per-choice price/quantity basis, gross, modeled cost, result total, and later actual values separately.
- **Testing implications:** Verify exact recomputation, unit consistency, unavailable components, and planned-versus-actual comparison.
- **Security / tenant-isolation implications:** FPO-specific economics and actual expense remain protected tenant data.
- **Related UNKNOWN / blocker references:** `CORE-004`, `CORE-005`, `CORE-007`, `PILOT-006`.
- **Explicit non-decisions:** Transport aggregation, actual-expense workflow, accounting treatment, rounding representation, and formula parameters are not selected.
- **Notes:** Use “expected net selling return,” not “profit.”

### IC-FAIRNESS-001 — Preserve versioned per-farmer leximin semantics exactly

- **Constraint family:** Policies, economics, and objectives
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §34, “Allocation fairness”; §35, “Leximin reference semantics”; §36, “MILP leximin representation”; §39, “Lexicographic objective preservation”
- **Architecture statement / rationale:** Current V1 fairness is true leximin of fulfilment ratios per participating farmer with positive participating quantity, not per lot or only the worst ratio; real FPO acceptance requires validation.
- **Technology-independent implementation obligation:** Express fairness as a versioned policy and preserve the exact business ordering/ratios independently of solver formulation.
- **Required invariant:** Each participating farmer has `P_f > 0`; all that farmer's participating lots contribute once to exact `S_f/P_f`; the complete sorted ratio vector is lexicographically maximized when this policy applies.
- **Failure semantics:** Solver tie/arbitrary named bottleneck assignment cannot redefine exact fairness truth; zero denominator is invalid preparation.
- **Audit / debugging implications:** Record participating farmer set, exact numerator/denominator, sorted objective tuple, policy version, and certified stages.
- **Testing implications:** Use tiny exact cases, equivalent bottlenecks, multiple lots per farmer, zero-denominator prevention, and differential formulations.
- **Security / tenant-isolation implications:** Farmer/lot aggregation cannot cross FPO boundaries.
- **Related UNKNOWN / blocker references:** `PILOT-009`.
- **Explicit non-decisions:** Production solver/formulation, operational acceptance, historical fairness, outcome fairness, and optimization encoding are not selected.
- **Notes:** The §36 MILP representation is an engineering formulation; only its stable semantics feed this constraint.

### IC-STORAGEPOLICY-001 — Apply storage priority without fabricated urgency or unbounded sacrifice

- **Constraint family:** Policies, economics, and objectives
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §37, “Storage objective”; §38, “Objective hierarchy”; §39, “Lexicographic objective preservation”
- **Architecture statement / rationale:** Current priority schedules `HIGH` then `MEDIUM` quantities/timing lexicographically, gives `LOW` no early pressure and `UNKNOWN` no fabricated rank, but requires an economic guardrail or hierarchy revision.
- **Technology-independent implementation obligation:** Preserve the stated storage objective semantics only under a validated economic guardrail and versioned StoragePolicy.
- **Required invariant:** `UNKNOWN` is never ranked as known; no deterioration trajectory is invented; preserved higher-stage optima cannot be sacrificed by lower stages.
- **Failure semantics:** Missing guardrail/operational validation prevents claiming the current hierarchy production-safe; it does not authorize arbitrary weights.
- **Audit / debugging implications:** Record category/basis, policy version, total and date-vector stage outcomes, preserved optima, and economic guardrail.
- **Testing implications:** Verify HIGH/MEDIUM order/timing, full-mode simplifications, `UNKNOWN`, lexicographic preservation, and guardrail gating.
- **Security / tenant-isolation implications:** Storage evidence and policy configuration remain FPO-scoped.
- **Related UNKNOWN / blocker references:** `CORE-005`, `PILOT-007`, `PILOT-009`, `PILOT-010`.
- **Explicit non-decisions:** Economic guardrail, trade-off behavior, thresholds, deterioration model, and policy implementation are not selected.
- **Notes:** None.

### IC-OBJECTIVE-001 — Preserve solver-independent lexicographic business priority

- **Constraint family:** Policies, economics, and objectives
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §38, “Objective hierarchy”; §39, “Lexicographic objective preservation”; §42, “OptimizationResult”
- **Architecture statement / rationale:** Technical/domain validity and economic admissibility bound the problem; allocation, fairness, storage, expected net return, and technical tie-break follow in lexicographic order, not arbitrary mega-weights.
- **Technology-independent implementation obligation:** Represent objective priorities as solver-independent sequential/lexicographic semantics and preserve every prior optimum before optimizing a lower priority.
- **Required invariant:** No lower-priority objective degrades a certified higher-priority objective; full mode skips only provably redundant stages.
- **Failure semantics:** Incomplete execution reports the certified objective prefix and cannot claim optimization of uncompleted stages.
- **Audit / debugging implications:** Record stage order, optimum/preservation result per stage, skipped-stage proof, and termination point.
- **Testing implications:** Verify conflicting-objective cases, partial/full mode stage sets, preservation, no arbitrary weighted blending, and early termination reporting.
- **Security / tenant-isolation implications:** N/A.
- **Related UNKNOWN / blocker references:** `CORE-005`, `PILOT-009`, `PILOT-010`.
- **Explicit non-decisions:** Solver objective API, encoding, weights, execution strategy, timeout, and gap policy are not selected.
- **Notes:** The hierarchy's production safety remains conditional where the architecture says so.

### IC-TIEBREAK-001 — Return a deterministic canonical representative after business ties

- **Constraint family:** Policies, economics, and objectives
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §40, “Deterministic tie-break”; §41, “OptimizationInput”; §44, “Solver verification”
- **Architecture statement / rationale:** After all business objectives tie, select a deterministic technical representative using the versioned canonical ordering, without introducing business preference.
- **Technology-independent implementation obligation:** Define and apply a reproducible technical tie-break only after all business objective equivalence is established.
- **Required invariant:** Repeated execution with the same exact input/business optimum returns the same canonical business-equivalent assignment.
- **Failure semantics:** Tie-break implementation cannot change any business objective or claim business meaning.
- **Audit / debugging implications:** Record tie-break definition/version, coordinate order, and pre-tie business tuple.
- **Testing implications:** Verify repeatability across equivalent optima and preservation of all business objectives.
- **Security / tenant-isolation implications:** N/A.
- **Related UNKNOWN / blocker references:** None.
- **Explicit non-decisions:** Efficient encoding, solver feature, call count, coefficient scheme, and canonical-ID representation are not selected.
- **Notes:** One-call-per-variable and giant exponential coefficients require benchmark evidence and are not selected.

## Optimization contracts

### IC-OPT-001 — Keep OptimizationEngine computationally isolated

- **Constraint family:** Optimization contracts
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §5, “Dependency direction”; §20, “TransportCostService”; §27, “Optimizer boundary”; §41, “OptimizationInput”
- **Architecture statement / rationale:** Optimization consumes validated solver-neutral input, returns a structured result, and performs no network calls or persistence writes.
- **Technology-independent implementation obligation:** Make OptimizationEngine a pure computational boundary independent of application framework, persistence, provider payloads, and network clients.
- **Required invariant:** The engine's only domain interaction is through OptimizationInput and OptimizationResult; it cannot mutate authoritative state.
- **Failure semantics:** Engine technical failure returns structured technical termination and creates no persistence/business side effect.
- **Audit / debugging implications:** Record exact input identity, engine implementation/version/config, result, and diagnostics outside the pure engine.
- **Testing implications:** Unit-test engine with in-memory exact inputs and verify absence of I/O/provider/database dependencies.
- **Security / tenant-isolation implications:** Input validation must establish tenant consistency before engine execution; engine receives no authorization bypass capability.
- **Related UNKNOWN / blocker references:** None.
- **Explicit non-decisions:** Solver/library, function signature, process isolation, serialization, and invocation mechanism are not selected.
- **Notes:** None.

### IC-OPT-002 — Distinguish technical admissibility, invalid input, and real infeasibility

- **Constraint family:** Optimization contracts
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §28, “Technical admissibility”; §47, “Validation layers”; §58, “Reliability philosophy”
- **Architecture statement / rationale:** `E_tech` contains technically usable choices; malformed/cross-domain input is invalid, provider outage is not real infeasibility, and storage `UNKNOWN` alone does not forbid a choice.
- **Technology-independent implementation obligation:** Build admissibility only from validated required evidence and report input invalidity, unavailable evidence, economic exclusion, and proven infeasibility as separate concepts.
- **Required invariant:** Every created choice is in the declared selected/candidate/date scope with usable required evidence; legitimate choices are not silently pruned for speed.
- **Failure semantics:** Provider/numeric/input failure never becomes business infeasibility, and absence from `E_tech` retains an explainable reason.
- **Audit / debugging implications:** Record inclusion/exclusion reason and evidence/status for each candidate coordinate or normalized reason class.
- **Testing implications:** Verify malformed scope, unavailable provider data, `UNKNOWN` storage, economic separation, sparse construction, and no false optimality after pruning.
- **Security / tenant-isolation implications:** Cross-FPO choices are invalid before solving.
- **Related UNKNOWN / blocker references:** `CORE-001`, `CORE-002`, `CORE-004`, `CORE-005`, `CORE-007`, `CORE-008`.
- **Explicit non-decisions:** Sparse data structure, route/forecast freshness rules, performance pruning, and implementation algorithm are not selected.
- **Notes:** None.

### IC-OPT-003 — Preserve assignment-variable physical meaning

- **Constraint family:** Optimization contracts
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §8, “Farmer and ProduceLot”; §29, “Core optimization variables”; §42, “OptimizationResult”
- **Architecture statement / rationale:** Planned `x[l,m,d]` is exact proposed physical quantity from a lot to mandi/date and may split across choices; it is not sale, reservation, storage, capacity, or a whole-lot indicator.
- **Technology-independent implementation obligation:** Keep internal solver representation separate while returning exact physical assignment quantities with their defined lot/mandi/date semantics.
- **Required invariant:** Every nonzero assignment is positive, exact, admissible, and contributes only to its lot's planned scheduled quantity.
- **Failure semantics:** Internal solver variable/scale cannot leak as physical quantity; semantic mismatch is output-invalid.
- **Audit / debugging implications:** Trace returned quantity to internal decision/reconstruction and canonical lot/mandi/date identities.
- **Testing implications:** Verify multi-mandi/day splits, exact conversion, no category confusion, and sparse result serialization.
- **Security / tenant-isolation implications:** Assignment lot and plan scope share the same FPO.
- **Related UNKNOWN / blocker references:** `CORE-006`.
- **Explicit non-decisions:** Solver variable type/encoding, data structure, scheduling increment, and DTO are not selected.
- **Notes:** None.

### IC-OPT-004 — Represent participating, scheduled, and remainder quantities explicitly

- **Constraint family:** Optimization contracts
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §30, “Participating quantity”; §34, “Allocation fairness”; §37, “Storage objective”; §67, “Revised/rejected ideas”
- **Architecture statement / rationale:** Participating quantity is positive and bounded by availability snapshot; partial mode permits explicit remainder, full mode requires equality, and increment incompatibility is detected before solving where possible.
- **Technology-independent implementation obligation:** Preserve selected/participating quantity, scheduled total, and unallocated remainder as distinct exact values under an explicit allocation policy.
- **Required invariant:** `0 < Q_plan <= Available_snapshot`, `S <= Q_plan`, `U = Q_plan - S`; full mode has `S = Q_plan`; partial remainder remains inventory and visible.
- **Failure semantics:** Incompatible full-mode quantity/increment is input/preflight failure, not silent rounding or unexplained infeasibility.
- **Audit / debugging implications:** Record allocation mode, snapshot availability, participating/scheduled/remainder per lot, and incompatibility reason.
- **Testing implications:** Verify full/partial cases, zero/over-allocation, remainder visibility, exact arithmetic, and incompatible increment.
- **Security / tenant-isolation implications:** Participating lots and availability snapshot remain FPO-consistent.
- **Related UNKNOWN / blocker references:** `CORE-006`.
- **Explicit non-decisions:** Default allocation mode, scheduling increment, UI display, and solver encoding are not selected.
- **Notes:** Protects `RR-001`.

### IC-OPT-005 — Keep OptimizationInput normalized, frozen, and solver-neutral

- **Constraint family:** Optimization contracts
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §25, “Revision-specific frozen evaluation basis”; §27, “Optimizer boundary”; §41, “OptimizationInput”; §46, “Numerical architecture”
- **Architecture statement / rationale:** OptimizationInput is a normalized mathematical projection of a richer frozen snapshot and excludes ORM/framework/provider/network objects; execution parameters are separate.
- **Technology-independent implementation obligation:** Produce a complete validated, ordered, unit-explicit, solver-neutral input from frozen evidence/policies, with execution configuration outside domain semantics.
- **Required invariant:** Identical frozen basis and normalization policy yield equivalent exact OptimizationInput; no mutable/external object is needed during solve.
- **Failure semantics:** Missing/invalid normalization or policy data is input/preparation failure before solving.
- **Audit / debugging implications:** Retain ordered sets, admissibility, quantities, ownership, prices, policies, transport math, normalization version, and source basis.
- **Testing implications:** Verify deterministic preparation, input validation, replay, unit exactness, forbidden object absence, and config/domain separation.
- **Security / tenant-isolation implications:** Preparation validates FPO consistency before producing input.
- **Related UNKNOWN / blocker references:** `CORE-004`, `CORE-005`, `CORE-006`, `CORE-007`, `CORE-008`.
- **Explicit non-decisions:** Data classes/DTOs, serialization, ORM, SQL schema, solver adapter, and execution parameter values are not selected.
- **Notes:** None.

### IC-OPT-006 — Report solution, proof, allocation, optimality, and termination independently

- **Constraint family:** Optimization contracts
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §42, “OptimizationResult”; §57, “OptimizationRun lifecycle vs outcome”; §58, “Reliability philosophy”
- **Architecture statement / rationale:** OptimizationResult is not a proposal/reservation/sale and separately reports solution availability, feasibility evidence, allocation outcome, optimality evidence, termination, exact assignments, and diagnostics.
- **Technology-independent implementation obligation:** Return solver-neutral structured evidence across independent axes and require downstream validation before proposal creation.
- **Required invariant:** Exact reconstructed assignments and business metrics are present for any candidate; mathematical zero-allocation feasibility does not automatically create a useful proposal.
- **Failure semantics:** Proven infeasible, not proven, time limit, technical failure, output-invalid, partial allocation, and no useful allocation cannot collapse into one status.
- **Audit / debugging implications:** Record objective prefix, proof/termination evidence, exact per-lot/farmer outcomes, economics, and diagnostics.
- **Testing implications:** Verify all axis combinations, early termination, zero allocation, invalid output, exact reconstruction, and independent PlanValidator gating.
- **Security / tenant-isolation implications:** Result identifiers must resolve only within input/plan FPO scope.
- **Related UNKNOWN / blocker references:** Proposal eligibility for feasible-not-fully-proven results remains an unregistered product-policy `UNKNOWN` in §42.
- **Explicit non-decisions:** Exact enums, proposal-eligibility policy, solver-native status mapping, serialization, and API contract are not selected.
- **Notes:** None.

### IC-OPT-007 — Keep the production solver replaceable and evidence-selected

- **Constraint family:** Optimization contracts
- **Architecture classification:** `ENGINEERING LEAN; DESIGN DECISION`
- **Source:** §36, “MILP leximin representation”; §43, “Solver strategy”; §44, “Solver verification”; §67, “Revised/rejected ideas”
- **Architecture statement / rationale:** MILP is the leading prototype formulation, while the engine remains replaceable and production selection follows correctness, status semantics, deployment/licensing, and validated-scale performance gates.
- **Technology-independent implementation obligation:** Prevent solver-library dependence in domain contracts and select any production engine only from reproducible evidence against the frozen business semantics.
- **Required invariant:** Replacing the engine cannot change OptimizationInput/Result domain meaning or validated business objective semantics.
- **Failure semantics:** Unsupported solver status/numerics is explicit adapter/technical failure; prototype success is not production selection.
- **Audit / debugging implications:** Record engine/library/version/config/licensing context, benchmark evidence, status mapping, and selection rationale.
- **Testing implications:** Differentially verify candidates against exact oracle cases and shared validators before selection.
- **Security / tenant-isolation implications:** N/A.
- **Related UNKNOWN / blocker references:** None.
- **Explicit non-decisions:** Production solver, library, modeling layer, formulation, timeout, gap, tolerance, seed, threads, and fallback are not selected.
- **Notes:** Protects `RR-009`; no ENGINEERING LEAN is upgraded.

### IC-OPT-008 — Independently verify optimization semantics and performance evidence

- **Constraint family:** Optimization contracts
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §35, “Leximin reference semantics”; §44, “Solver verification”; §62, “Testing strategy”
- **Architecture statement / rationale:** Tiny brute-force/oracle cases, differential solvers, independent PlanValidator, complete objective tuples, failure injection, and reproducible benchmarks verify the engine.
- **Technology-independent implementation obligation:** Maintain an implementation-independent correctness oracle/validation path and collect comparable performance/proof evidence at validated scales.
- **Required invariant:** A candidate result cannot be trusted solely because the solver reports success; exact validator/oracle semantics decide domain correctness.
- **Failure semantics:** Solver-oracle/validator disagreement is a correctness failure and blocks selection/proposal, not a benchmark variance.
- **Audit / debugging implications:** Retain cases, seeds/configs, dimensions/density, build/solve/validation time, memory, termination/proof, objective tuple, and comparison result.
- **Testing implications:** Tiny exhaustive cases, differential engines, repeatability, frozen replay, failure injection, and full objective-tuple comparisons are required.
- **Security / tenant-isolation implications:** Test fixtures must include FPO-isolation cases and protect any real data used for benchmarks.
- **Related UNKNOWN / blocker references:** Validated FPO scale remains evidence-dependent.
- **Explicit non-decisions:** Test framework, benchmark tooling, production solver, scenario size caps, and performance thresholds are not selected.
- **Notes:** None.

### IC-OPT-009 — Preserve problem semantics under scalability work

- **Constraint family:** Optimization contracts
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §28, “Technical admissibility”; §36, “MILP leximin representation”; §40, “Deterministic tie-break”; §45, “Scalability”
- **Architecture statement / rationale:** Allocation width follows admissible choices; fairness/objective depth may grow; unvalidated caps, silent pruning, or unproven decomposition cannot claim equivalent/global optimum.
- **Technology-independent implementation obligation:** Make all scalability transformations explicit and prove or disclose their effect on feasible region, objectives, and optimality claims.
- **Required invariant:** An optimization result's claims match the exact problem actually solved; no legitimate choice is omitted without recorded semantics.
- **Failure semantics:** Resource limit/approximation is reported as limited proof/changed problem, not global optimum for the original problem.
- **Audit / debugging implications:** Record dimensions, density, transformations/pruning, model size, objective stages, resource limits, and proof status.
- **Testing implications:** Compare transformed/sparse models with exact small originals and benchmark validated representative scales.
- **Security / tenant-isolation implications:** N/A.
- **Related UNKNOWN / blocker references:** Real FPO scale and transport grouping remain unresolved evidence inputs.
- **Explicit non-decisions:** Production caps, decomposition, heuristic, sparse structure, solver, tie-break encoding, and performance target are not selected.
- **Notes:** None.

## Human agency and information provenance (continued)

### IC-PROVENANCE-001 — Preserve fact, derivation, decision, and reality layers

- **Constraint family:** Human agency and information provenance
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §3, “Fundamental architecture rule”; §25, “Revision-specific frozen evaluation basis”; §53, “Planned vs actual execution”; §71, “Final mental model”
- **Architecture statement / rationale:** Raw/authoritative facts, derived information, system decisions, human decisions, and reality are different layers and must never overwrite one another.
- **Technology-independent implementation obligation:** Represent and retain each layer independently with traceable relationships between source facts, derived values, proposals, approvals, and actual outcomes.
- **Required invariant:** A value or event in one layer cannot replace or masquerade as a value or event in another layer.
- **Failure semantics:** Missing or changed derived evidence must not rewrite historical facts, decisions, or actual outcomes.
- **Audit / debugging implications:** A result must be reconstructable from its facts, derived versions, system output, human action, and subsequent reality.
- **Testing implications:** Verify historical reconstruction and that updates to one layer do not mutate another.
- **Security / tenant-isolation implications:** Traceability records associated with tenant-owned objects inherit FPO isolation.
- **Related UNKNOWN / blocker references:** `CORE-009`, `PILOT-006`, `PILOT-011`.
- **Explicit non-decisions:** Persistence layout, event-storage mechanism, schema, and serialization are not selected.
- **Notes:** This is the strongest system-wide invariant.

## Module and dependency boundaries

### IC-MODULE-001 — Preserve a modular monolith application boundary

- **Constraint family:** Module and dependency boundaries
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §4, “Application shape”
- **Architecture statement / rationale:** V1 is a modular monolith with logical domain modules; unnecessary distributed infrastructure is excluded unless later justified.
- **Technology-independent implementation obligation:** Keep V1 within one cohesive application boundary while maintaining explicit logical ownership and interfaces between concerns.
- **Required invariant:** Domain responsibilities remain modular even though they share the V1 application/deployment boundary.
- **Failure semantics:** A cross-module failure must not be hidden by duplicating or scattering domain ownership.
- **Audit / debugging implications:** Module ownership and call paths must be identifiable.
- **Testing implications:** Verify module contracts independently and integration across their declared boundaries.
- **Security / tenant-isolation implications:** Shared application deployment must not weaken server-side tenant isolation.
- **Related UNKNOWN / blocker references:** None.
- **Explicit non-decisions:** Directory structure, framework versions, packaging, process model, and deployment platform are not selected; architecture-level product names are not version selections.
- **Notes:** No microservice, broker, cache, or orchestration infrastructure is introduced by this constraint.

### IC-MODULE-002 — Enforce inward dependency flow and stable domain interfaces

- **Constraint family:** Module and dependency boundaries
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §5, “Dependency direction”; §19, “Routing”; §21, “External provider pattern”; §27, “Optimizer boundary”
- **Architecture statement / rationale:** Provider adapters normalize into domain services, planning produces solver-neutral input, and the optimizer remains isolated from frameworks, persistence, and providers.
- **Technology-independent implementation obligation:** Dependencies must point from external adapters toward normalized domain boundaries and from orchestration toward pure computational contracts, without reverse coupling or circular ownership.
- **Required invariant:** Provider-specific objects, framework objects, persistence sessions, and network clients never become cross-domain or optimizer contracts.
- **Failure semantics:** Adapter/provider failure is normalized at the boundary and must not leak provider-specific interpretation into domain behavior.
- **Audit / debugging implications:** Each external-to-domain normalization and downstream dependency edge must be traceable.
- **Testing implications:** Substitute adapters and optimization engines through stable interfaces; detect forbidden dependency direction and circular ownership.
- **Security / tenant-isolation implications:** External adapters cannot bypass authoritative domain authorization boundaries.
- **Related UNKNOWN / blocker references:** None.
- **Explicit non-decisions:** Module layout, dependency-injection mechanism, interface syntax, and provider SDK are not selected.
- **Notes:** None.

## Tenancy and authorization

### IC-TENANCY-001 — Authoritative multi-FPO isolation

- **Constraint family:** Tenancy and authorization
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §6, “Multi-FPO”; §47, “Validation layers”; §59, “Idempotency and concurrency”; §62, “Testing strategy”
- **Architecture statement / rationale:** All tenant-owned business objects are unambiguously FPO-scoped and server-side authorization enforces isolation; Mandi may be global while FPO configuration remains scoped.
- **Technology-independent implementation obligation:** Every authoritative read, mutation, validation, and concurrent operation on tenant-owned resources must enforce the principal's authorized FPO boundary.
- **Required invariant:** A principal cannot access, relate, reserve, approve, sell, configure, or mutate tenant-owned data outside the authorized FPO; global reference data cannot collapse tenant-scoped configuration.
- **Failure semantics:** Cross-FPO input or reference is authorization/input-invalid failure, never a valid empty result or masked optimizer choice.
- **Audit / debugging implications:** Record tenant context and authorization outcome for material operations.
- **Testing implications:** Verify cross-FPO reads/writes/relationships fail, including randomized and concurrent flows.
- **Security / tenant-isolation implications:** This constraint is itself the primary tenant-isolation requirement; enforcement must be authoritative and server-side.
- **Related UNKNOWN / blocker references:** `PILOT-003`.
- **Explicit non-decisions:** Tenant-key representation, row/collection strategy, authorization library, database policy, and schema are not selected.
- **Notes:** Do not hard-code a demo FPO.

### IC-TENANCY-002 — Flexible evidence-based roles and permissions

- **Constraint family:** Tenancy and authorization
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §7, “Actors”; §49, “Approval flow”; §64, “Real-FPO unknowns — must validate before pilot”
- **Architecture statement / rationale:** The primary V1 web actor is FPO manager/staff, but exact operational roles and authorities remain unknown.
- **Technology-independent implementation obligation:** Authorization boundaries must permit validated roles and authorities to be defined later without encoding privileges from identity labels such as email or name.
- **Required invariant:** No unvalidated role or privilege becomes production authority.
- **Failure semantics:** Unknown or unauthorized authority must block the protected action explicitly, not infer permission from identity attributes.
- **Audit / debugging implications:** Record principal, resolved authority, tenant, action, and denial reason.
- **Testing implications:** Verify configurable role mappings, denial by default for unresolved authority, and no privilege inference from names/emails.
- **Security / tenant-isolation implications:** Role flexibility cannot weaken FPO isolation or authoritative server enforcement.
- **Related UNKNOWN / blocker references:** `PILOT-002`, `PILOT-003`, `PILOT-004`, `PILOT-005`.
- **Explicit non-decisions:** Role names, permission matrix, authentication method, token/session mechanism, and administrative workflow are not selected.
- **Notes:** None.

## Domain identity and physical model

### IC-DOMAIN-001 — Model produce as farmer-owned lots

- **Constraint family:** Domain identity and physical model
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §8, “Farmer and ProduceLot”; §29, “Core optimization variables”; §54, “ActualSale”
- **Architecture statement / rationale:** Crop/quantity belongs to farmer-owned ProduceLots rather than Farmer; a lot is a commodity batch, physical splitting uses child/sub-lots, and planned allocation splitting does not physically split it.
- **Technology-independent implementation obligation:** Preserve FarmerMember-to-ProduceLot ownership, batch identity, commodity identity, and the distinction between physical sub-lots and multiple planned assignments.
- **Required invariant:** Every lot has one owning member and commodity batch identity; assignments do not mutate physical lot identity; ActualSale remains anchored to a ProduceLot.
- **Failure semantics:** Conflicting ownership/commodity or treating an assignment as a physical split is input/domain invalidity.
- **Audit / debugging implications:** Preserve lot lineage, ownership, physical splits, and assignment references.
- **Testing implications:** Verify multiple lots/commodities per farmer, child-lot lineage, and multi-assignment without unintended physical split.
- **Security / tenant-isolation implications:** FarmerMember, lot, child lot, assignment, and sale relationships must remain within one FPO.
- **Related UNKNOWN / blocker references:** `CORE-007`, `PILOT-001`, `PILOT-002`.
- **Explicit non-decisions:** Entity storage, identifiers, quality fields, registration workflow, and split operation mechanism are not selected.
- **Notes:** None.

### IC-DOMAIN-002 — Canonical internal identity across modules

- **Constraint family:** Domain identity and physical model
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §23, “Canonical identifiers”; §40, “Deterministic tie-break”
- **Architecture statement / rationale:** Internal entities use canonical IDs; provider names/IDs map at adapters and raw strings do not serve as cross-module identity.
- **Technology-independent implementation obligation:** Use stable canonical identity for cross-module references and retain explicit mappings/provenance for external identifiers.
- **Required invariant:** Equivalent domain objects cannot diverge solely because provider labels differ, and provider identifiers cannot replace canonical identity.
- **Failure semantics:** Missing or ambiguous mapping is explicit unavailable/invalid data, not an implicit string match.
- **Audit / debugging implications:** Mapping source, external identifier, canonical target, and mapping history must be inspectable where relevant.
- **Testing implications:** Verify provider-label variation maps deterministically or fails explicitly without cross-entity collision.
- **Security / tenant-isolation implications:** Canonical references must preserve global-versus-FPO ownership rules.
- **Related UNKNOWN / blocker references:** `CORE-007`.
- **Explicit non-decisions:** Identifier type, generation method, mapping persistence, and schema are not selected.
- **Notes:** Variety/grade remains extensible until validated.

### IC-DOMAIN-003 — Bound each SellingPlan's planning problem explicitly

- **Constraint family:** Domain identity and physical model
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §22, “SellingPlan”; §28, “Technical admissibility”
- **Architecture statement / rationale:** A V1 SellingPlan has one FPO, one commodity, finite dates, selected eligible lots, candidate mandis, and policy; eligible is not selected and candidate is not assigned.
- **Technology-independent implementation obligation:** Represent the manager-selected planning boundary explicitly and preserve each distinction when preparing, validating, and explaining a plan.
- **Required invariant:** A plan cannot mix FPOs or commodities; only selected lots and candidate mandis enter the declared search boundary; selection/candidacy never implies assignment.
- **Failure semantics:** Cross-FPO/commodity or outside-boundary choices are input invalidity, not silently pruned coordinates.
- **Audit / debugging implications:** Record the eligible set, selected set, candidate set, date window, commodity, FPO, and policy versions.
- **Testing implications:** Verify boundary distinctions and rejection of choices outside the declared problem.
- **Security / tenant-isolation implications:** Plan boundary and all selected tenant objects must belong to the same authorized FPO.
- **Related UNKNOWN / blocker references:** `CORE-008`.
- **Explicit non-decisions:** UI selection flow, request DTO, persistence model, and future multi-commodity batching are not selected.
- **Notes:** Multiple commodities use separate V1 plans.

### IC-DOMAIN-004 — Preserve physical placement and device-assignment history

- **Constraint family:** Domain identity and physical model
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §15, “Storage physical model”; §63, “Real-FPO unknowns — priority classification”
- **Architecture statement / rationale:** A ProduceLot has one active physical placement at an instant with history; device assignment also has history so readings remain interpretable after movement.
- **Technology-independent implementation obligation:** Preserve temporally interpretable lot placement and device-to-zone assignment without overwriting prior placement/assignment facts.
- **Required invariant:** At an instant, a non-physically-split lot has one active placement; historical readings resolve the device assignment valid when recorded.
- **Failure semantics:** Ambiguous overlapping active placement/assignment is explicit invalid state, not guessed location.
- **Audit / debugging implications:** Reconstruct where a lot and device were assigned at any relevant time.
- **Testing implications:** Verify moves close prior active history, old readings retain original context, and physical child-lots can carry separate placements.
- **Security / tenant-isolation implications:** Tenant-owned placements/devices cannot cross FPO scope unintentionally.
- **Related UNKNOWN / blocker references:** `CORE-001`, `CORE-002`, `PILOT-008`.
- **Explicit non-decisions:** Physical custody answer, location schema, temporal representation, sensor multiplicity, and movement workflow are not selected.
- **Notes:** None.

## Numeric semantics

### IC-NUMERIC-001 — Exact quantity semantics with distinct quanta

- **Constraint family:** Numeric semantics
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §9, “Quantity representation”; §29, “Core optimization variables”; §30, “Participating quantity”; §46, “Numerical architecture”
- **Architecture statement / rationale:** Authoritative quantities use exact arithmetic; canonical unit, planning increment, and solver normalization quantum are distinct; ActualSale is not constrained by planning increment.
- **Technology-independent implementation obligation:** Preserve exact physical quantity semantics across domain, planning, solver normalization, reconstruction, and actual sales without conflating the three quantity concepts.
- **Required invariant:** No authoritative inventory quantity is represented through ordinary binary floating-point approximation; solver normalization is lossless; ActualSale truth is not rounded to a planning increment.
- **Failure semantics:** Non-lossless conversion or incompatible full-mode increment is explicit preparation/input failure, never silent rounding or business infeasibility.
- **Audit / debugging implications:** Record unit, increment, normalization quantum, conversion/version, and exact reconstructed quantity.
- **Testing implications:** Verify exact round trips, boundary quantities, incompatible increments, and no hidden rounding.
- **Security / tenant-isolation implications:** N/A.
- **Related UNKNOWN / blocker references:** `CORE-006`.
- **Explicit non-decisions:** Canonical unit, schedulable increment, numeric type, scale, storage type, and solver encoding are not selected.
- **Notes:** The architecture's kilogram/quintal values are examples/questions, not frozen answers.

### IC-NUMERIC-002 — Exact money and rate semantics

- **Constraint family:** Numeric semantics
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §10, “Money and rate representation”; §18, “Effective optimizer price”; §33, “Expected net selling return”; §47, “Validation layers”
- **Architecture statement / rationale:** Money and rates are distinct exact business values; ML floats are normalized deterministically/versionedly; authoritative arithmetic has no hidden intermediate rounding.
- **Technology-independent implementation obligation:** Preserve exact, unit-aware money/rate calculations from normalized forecast through validation and expected-net-return reconstruction, separating display rounding from authority.
- **Required invariant:** Business money/rate truth never depends on ordinary binary floating-point approximation or unrecorded rounding; units cannot be combined incompatibly.
- **Failure semantics:** Invalid unit/scale or non-deterministic normalization is explicit input/numeric failure.
- **Audit / debugging implications:** Record source value, unit, normalization policy/version, exact intermediate values, and display transformation separately.
- **Testing implications:** Verify exact arithmetic, unit compatibility, deterministic normalization, and display/authority separation.
- **Security / tenant-isolation implications:** N/A.
- **Related UNKNOWN / blocker references:** `CORE-004`, `CORE-005`, `CORE-007`, `PILOT-006`.
- **Explicit non-decisions:** Currency storage, decimal scale, rounding rule values, SQL type, and forecast-normalization algorithm are not selected.
- **Notes:** Expected net selling return is not labelled profit.

### IC-NUMERIC-003 — Lossless solver-domain reconstruction

- **Constraint family:** Numeric semantics
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §35, “Leximin reference semantics”; §36, “MILP leximin representation”; §42, “OptimizationResult”; §46, “Numerical architecture”
- **Architecture statement / rationale:** Solver numerics are not domain truth; exact quantities and fairness ratios are reconstructed/recomputed, with domain-specific numerical policies rather than a universal EPS.
- **Technology-independent implementation obligation:** Treat solver numeric output as provisional evidence and reconstruct exact domain decisions before validation, inventory, reservation, or fairness use.
- **Required invariant:** Raw solver floating output never becomes authoritative quantity or fairness truth; every reconstructed decision satisfies exact domain representation.
- **Failure semantics:** Reconstruction/preflight failure is technical failure, not business infeasibility; tolerance acceptance cannot authorize invalid domain quantity.
- **Audit / debugging implications:** Record solver/version/config, raw result diagnostics, reconstruction steps, normalization policy, and exact recomputation.
- **Testing implications:** Verify tolerance-edge reconstruction, NaN/infinity/overflow/coefficient preflight, exact fairness ratios, and rejection of non-reconstructable output.
- **Security / tenant-isolation implications:** N/A.
- **Related UNKNOWN / blocker references:** `CORE-006`.
- **Explicit non-decisions:** Solver, tolerance values, EPS values, numeric library, and encoding are not selected.
- **Notes:** The MILP formulation remains an engineering formulation, not a production solver choice.

## Inventory and reservations

### IC-INVENTORY-001 — Derive availability from authoritative quantities

- **Constraint family:** Inventory and reservations
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §11, “Inventory architecture”; §12, “QuantityReservation”; §14, “Off-plan ActualSale vs active reservation”
- **Architecture statement / rationale:** Availability is authoritative lot quantity minus actual sold, active remaining reservations, and finalized adjustments; editable available quantity is forbidden.
- **Technology-independent implementation obligation:** Compute availability from authoritative components under exact arithmetic rather than maintaining an independently mutable availability fact.
- **Required invariant:** Computed availability equals the architecture formula and never becomes negative.
- **Failure semantics:** Component inconsistency or negative availability is explicit invariant violation, not clamped or hidden.
- **Audit / debugging implications:** Expose each authoritative component and the calculation basis at the relevant point in time.
- **Testing implications:** Verify derivation across reserve/release/sell/adjust/split sequences and randomized histories.
- **Security / tenant-isolation implications:** All contributing quantities must belong to the same FPO and lot lineage.
- **Related UNKNOWN / blocker references:** `CORE-009`.
- **Explicit non-decisions:** Persistence model, query strategy, caching, and database constraint representation are not selected.
- **Notes:** None.

### IC-INVENTORY-002 — Centralize auditable inventory mutation

- **Constraint family:** Inventory and reservations
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §11, “Inventory architecture”; §59, “Idempotency and concurrency”; §61, “Audit and feedback”
- **Architecture statement / rationale:** Inventory mutations pass through one domain boundary and finalized changes are transactional, auditable, idempotent where required, and correctable without deleting history.
- **Technology-independent implementation obligation:** Route every authoritative inventory change through a single governed mutation boundary that enforces exact invariants and records the change history.
- **Required invariant:** No code path independently decrements/increments lot truth outside the inventory boundary; corrections preserve the superseded historical fact.
- **Failure semantics:** Partial mutation must not be committed; duplicate requests must not repeat an already-applied effect.
- **Audit / debugging implications:** Record actor/event, operation, before/after components, reason, idempotency identity where applicable, and correction lineage.
- **Testing implications:** Verify each mutation operation, rollback/atomicity, duplicate handling, correction history, and forbidden bypasses.
- **Security / tenant-isolation implications:** Mutation authorization and every referenced object are FPO-scoped.
- **Related UNKNOWN / blocker references:** `PILOT-002`, `PILOT-003`, `PILOT-005`.
- **Explicit non-decisions:** Service syntax, transaction engine, locking mechanism, storage model, and correction workflow details are not selected.
- **Notes:** None.

### IC-INVENTORY-003 — Approval-created reservation accounting

- **Constraint family:** Inventory and reservations
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §12, “QuantityReservation”; §49, “Approval flow”; §59, “Idempotency and concurrency”
- **Architecture statement / rationale:** Proposals do not reserve; approval atomically rechecks availability, creates reservations, and approves the exact revision. Reservation state derives from reserved, consumed, and released quantities.
- **Technology-independent implementation obligation:** Create commitment only during authorized approval and maintain exact quantity accounting for reservation consumption and release.
- **Required invariant:** `remaining_reserved = reserved - consumed - released`, all components are non-negative, and one approval request cannot reserve twice.
- **Failure semantics:** Failed approval creates neither approval nor reservation; duplicate approval is idempotent; display state cannot override quantity truth.
- **Audit / debugging implications:** Link reservation quantities to the exact approved revision/assignment and record consume/release operations.
- **Testing implications:** Verify proposal non-reservation, atomic approval, partial consumption/release, derived states, and concurrent/double-click approval.
- **Security / tenant-isolation implications:** Only an authorized actor in the owning FPO can approve or mutate its reservation.
- **Related UNKNOWN / blocker references:** `PILOT-003`, `PILOT-004`.
- **Explicit non-decisions:** Locking, isolation level, idempotency store, database constraints, and approval API are not selected.
- **Notes:** None.

### IC-INVENTORY-004 — Quantity-aware cancellation preserving approval history

- **Constraint family:** Inventory and reservations
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §13, “Cancellation modeling”; §50, “PlanRevision states”; §51, “Revision vs replan”; §67, “Revised/rejected ideas”
- **Architecture statement / rationale:** Approval remains a historical decision while cancellation/release operates on remaining commitments; executed ActualSales are never undone.
- **Technology-independent implementation obligation:** Represent post-approval cancellation as auditable quantity-aware release/cancellation of unexecuted commitment without erasing or mutating the historical approval.
- **Required invariant:** Released/cancelled quantity cannot exceed remaining unexecuted reservation, and executed sales plus historical approval remain immutable facts.
- **Failure semantics:** Cancellation cannot reverse reality or reopen an approved revision; partial cancellation cannot masquerade as universal revision cancellation.
- **Audit / debugging implications:** Retain original approval, assignment-level cancellation/release quantities, actor/reason, and derived aggregate cancellation.
- **Testing implications:** Verify partial/full remaining release, post-partial-execution cancellation, history preservation, and no effect on executed sales.
- **Security / tenant-isolation implications:** Cancellation/release authority remains FPO-scoped and auditable.
- **Related UNKNOWN / blocker references:** `PILOT-003`.
- **Explicit non-decisions:** Cancellation reason codes, authorization matrix, aggregate UI labels, and persistence mechanism are not selected.
- **Notes:** Protects `RR-002` and `RR-007`.

### IC-INVENTORY-005 — Preserve lot inventory during sale/reservation reconciliation

- **Constraint family:** Inventory and reservations
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §14, “Off-plan ActualSale vs active reservation”; §54, “ActualSale”; §55, “Execution state”; §63, “Real-FPO unknowns — priority classification”
- **Architecture statement / rationale:** ActualSale remains recordable, linked sales consume the corresponding reservation, and total actual sold plus remaining commitments/decrements cannot exceed authoritative quantity; the off-plan reconciliation workflow is unresolved.
- **Technology-independent implementation obligation:** Record reality transactionally while preserving lot-level inventory and never arbitrarily selecting an active reservation to break.
- **Required invariant:** `actual sold + remaining active reservations + other finalized decrements <= authoritative lot quantity` after every committed operation.
- **Failure semantics:** Do not reject reality merely because a plan exists, silently steal from a reservation, or commit over-allocation; unresolved off-plan conflict requires explicit auditable resolution.
- **Audit / debugging implications:** Record sale linkage, reservation consumption, deviations, unresolved conflict context, and any later reconciliation operation.
- **Testing implications:** Verify linked-sale consumption, concurrent sale/reservation conflict, over-plan deviations, and—only after policy resolution—off-plan reconciliation.
- **Security / tenant-isolation implications:** Sale and reservation references must share lot/FPO scope; recording/reconciliation authority remains unresolved.
- **Related UNKNOWN / blocker references:** `CORE-009`, `PILOT-005`.
- **Explicit non-decisions:** Off-plan reconciliation workflow, reservation release order, actor authority, locking, and transaction mechanism are not selected.
- **Notes:** The unknown blocks final production `record_sale()` behavior for this case.

## Storage and IoT

### IC-STORAGE-001 — Conservative storage assessment from bounded sensor facts

- **Constraint family:** Storage and IoT
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §3, “Fundamental architecture rule”; §16, “IoT / StorageAssessment”; §58, “Reliability philosophy”; §67, “Revised/rejected ideas”
- **Architecture statement / rationale:** Ambient temperature/humidity readings are not grain moisture, quality, or spoilage; StorageAssessment is a separate derived value with `HIGH`, `MEDIUM`, `LOW`, and mandatory `UNKNOWN`.
- **Technology-independent implementation obligation:** Derive conservative, provenance-bearing storage assessments without overstating sensor meaning or converting missing/stale/invalid/offline data into a safe category.
- **Required invariant:** SensorReading remains distinct from StorageAssessment; unavailable/unusable evidence yields `UNKNOWN`, never fabricated `LOW` or spoilage truth.
- **Failure semantics:** Device/provider failure is explicit health/data unavailability, not storage-condition evidence.
- **Audit / debugging implications:** Retain readings, validity/freshness, device health, threshold/policy version, and resulting assessment separately.
- **Testing implications:** Verify boundary categories only after policy validation, all unavailable cases, stale data, invalid readings, and no unsupported spoilage assertion.
- **Security / tenant-isolation implications:** Device, zone, reading, lot placement, and assessment relationships must respect FPO ownership.
- **Related UNKNOWN / blocker references:** `PILOT-007`, `PILOT-008`, `PILOT-010`.
- **Explicit non-decisions:** Storage thresholds, sampling rate, sensor placement, MQTT configuration, urgency meaning, and scientific model are not selected.
- **Notes:** Protects `RR-004`.

### IC-STORAGE-002 — Keep storage policy replaceable and uncertainty-aware

- **Constraint family:** Storage and IoT
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §16, “IoT / StorageAssessment”; §25, “Revision-specific frozen evaluation basis”; §37, “Storage objective”; §48, “Proposal freshness”
- **Architecture statement / rationale:** Storage assessments/policies are versioned evidence, use only current frozen assessment, and cannot invent deterioration trajectories; operational/scientific thresholds remain unknown.
- **Technology-independent implementation obligation:** Evaluate storage urgency through an explicit versioned boundary whose evidence can be frozen per revision and later replaced without rewriting history.
- **Required invariant:** A revision uses the recorded assessment/policy version; later sensor/policy change cannot rewrite its historical evaluation.
- **Failure semantics:** Unknown or stale assessment remains explicit and receives no fabricated rank/trajectory.
- **Audit / debugging implications:** Record assessment identity, source readings, generated time, category, freshness, and policy/threshold version.
- **Testing implications:** Verify replay against frozen assessment, version changes, `UNKNOWN` handling, and no implicit deterioration.
- **Security / tenant-isolation implications:** Frozen storage evidence must correspond to the plan's FPO/lot.
- **Related UNKNOWN / blocker references:** `PILOT-007`, `PILOT-008`, `PILOT-010`, `PILOT-011`.
- **Explicit non-decisions:** Thresholds, deterioration model, multiple-sensor aggregation, freshness limits, and policy implementation are not selected.
- **Notes:** None.

## Forecasting

### IC-FORECAST-001 — Separate offline training from production inference

- **Constraint family:** Forecasting
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §17, “Forecasting”; §61, “Audit and feedback”
- **Architecture statement / rationale:** Training produces versioned artifacts offline; production inference uses an artifact/current input and does not retrain per request or directly from feedback.
- **Technology-independent implementation obligation:** Maintain distinct training, artifact-versioning, inference, and human-governed feedback-to-model processes.
- **Required invariant:** A production inference identifies its model artifact/version and cannot trigger implicit retraining.
- **Failure semantics:** Missing/unusable artifact or inference failure produces unavailable forecast, not automatic training or a fabricated price.
- **Audit / debugging implications:** Record artifact/version, inputs, generation time, status, and the human-controlled lineage of model updates.
- **Testing implications:** Verify artifact selection, reproducible inference contract, unavailable states, and absence of request-time/feedback-triggered retraining.
- **Security / tenant-isolation implications:** Training/inference data access must respect data ownership and authorized scope where tenant data is used.
- **Related UNKNOWN / blocker references:** None.
- **Explicit non-decisions:** ML framework, final model, training schedule, artifact store, feature set, weather use, and deployment mechanism are not selected.
- **Notes:** A3T-GCN remains a candidate, not a dependency.

### IC-FORECAST-002 — Normalize forecasts behind a future-capable effective-price boundary

- **Constraint family:** Forecasting
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §10, “Money and rate representation”; §17, “Forecasting”; §18, “Effective optimizer price”; §23, “Canonical identifiers”; §67, “Revised/rejected ideas”
- **Architecture statement / rationale:** Domain PriceForecast is normalized by mandi/commodity/date with provenance/version/status, while optimizer input accepts choice-level effective `p[l,m,d]`; unvalidated quality effects are not fabricated.
- **Technology-independent implementation obligation:** Keep model/provider internals outside the normalized forecast contract and derive effective choice-level price through an explicit versioned transformation that can later incorporate validated lot factors.
- **Required invariant:** Every effective price traces to normalized forecast evidence and policy; V1 mapping does not imply permanent lot-independence or quality irrelevance.
- **Failure semantics:** Missing/invalid forecast or mapping is unavailable/invalid choice evidence, never zero price.
- **Audit / debugging implications:** Record forecast ID/value/unit/model version/generated time/status and effective-price transformation/version.
- **Testing implications:** Verify canonical identity mapping, deterministic normalization, V1 direct mapping, unavailable status, and future lot-level substitution without contract breakage.
- **Security / tenant-isolation implications:** FPO-specific selection/configuration around global market data remains scoped.
- **Related UNKNOWN / blocker references:** `CORE-007`.
- **Explicit non-decisions:** Model, provider, quality adjustment, numeric representation, data source, confidence usage, and DTO/schema are not selected.
- **Notes:** Protects `RR-008`.

## Providers, routing, and transport

### IC-PROVIDER-001 — Normalize external-provider outcomes with provenance

- **Constraint family:** Providers, routing, and transport
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §19, “Routing”; §21, “External provider pattern”; §58, “Reliability philosophy”
- **Architecture statement / rationale:** Provider calls use bounded failure handling, validation, normalization, and source/status/time metadata; fabricated zero fallbacks are forbidden.
- **Technology-independent implementation obligation:** Place every external interaction behind an adapter that produces validated domain outcomes with explicit provenance, freshness, and unavailable/error states.
- **Required invariant:** No provider failure or missing value becomes a believable domain zero or unrelated real-world condition.
- **Failure semantics:** Timeout/retry exhaustion/invalid response is explicit technical unavailability/error, distinct from domain infeasibility.
- **Audit / debugging implications:** Record request correlation, provider/source, timestamps, validation result, retry/termination information, and normalized status.
- **Testing implications:** Contract-test valid/invalid/time-out/stale responses, retry safety, normalization, and zero-fallback prohibition.
- **Security / tenant-isolation implications:** Credentials and provider calls cannot bypass tenant authorization or expose cross-FPO data.
- **Related UNKNOWN / blocker references:** `PILOT-008`, `PILOT-011`.
- **Explicit non-decisions:** Provider, SDK, timeout/retry values, transport protocol, cache, queue, and credential mechanism are not selected.
- **Notes:** Retries apply only where safe.

### IC-ROUTING-001 — Provider-independent route estimation

- **Constraint family:** Providers, routing, and transport
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §19, “Routing”; §20, “TransportCostService”; §67, “Revised/rejected ideas”
- **Architecture statement / rationale:** RoutingService returns normalized distance/duration/status/provenance/freshness and is replaceable; it does not compute FPO-specific rupee cost.
- **Technology-independent implementation obligation:** Expose route estimates through a provider-neutral domain boundary and keep route evidence separate from transport economics.
- **Required invariant:** A route result identifies normalized distance/duration and provenance/status; provider objects and rupee cost do not enter the route contract.
- **Failure semantics:** Provider failure is unavailable/error, not road blockage or zero distance.
- **Audit / debugging implications:** Preserve provider/source, request endpoints/canonical mappings, estimate values/units, timestamp, and status.
- **Testing implications:** Verify provider substitution, normalized units/status, failure distinctions, and absence of transport-cost leakage.
- **Security / tenant-isolation implications:** FPO-specific locations/configuration and route access remain authorized.
- **Related UNKNOWN / blocker references:** `CORE-001`, `CORE-002`.
- **Explicit non-decisions:** Routing provider, API, SDK, cache/freshness policy, origin workflow, and network mechanism are not selected.
- **Notes:** Protects `RR-005`.

### IC-TRANSPORT-001 — Isolate deterministic transport economics from routing and solving

- **Constraint family:** Providers, routing, and transport
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §20, “TransportCostService”; §27, “Optimizer boundary”; §33, “Expected net selling return”; §41, “OptimizationInput”
- **Architecture statement / rationale:** Transport economics are distinct from routes, are frozen/deterministic for optimization, perform no network calls in the optimizer, and must remain extensible for validated cost structures.
- **Technology-independent implementation obligation:** Normalize transport evidence into a versioned deterministic model before optimization while preserving estimated-versus-actual expense and future structural extensibility.
- **Required invariant:** Optimization uses only frozen transport economics; route estimates do not directly become rupee costs; actual expense cannot overwrite the estimate.
- **Failure semantics:** Missing rate/quote/model is unavailable technical/evidence state, not zero cost; arbitrary large activation bounds are invalid preparation.
- **Audit / debugging implications:** Record source, assumptions/parameters, units, version, quote validity where applicable, modeled cost, and later actual expense separately.
- **Testing implications:** Verify determinism, no network calls in solving, unit/evidence failures, estimated/actual separation, and future fixed/variable compatibility once policy exists.
- **Security / tenant-isolation implications:** FPO-specific transport evidence/configuration and quotes remain scoped.
- **Related UNKNOWN / blocker references:** `CORE-001`, `CORE-002`, `CORE-003`, `CORE-004`, `PILOT-006`.
- **Explicit non-decisions:** Formula, dispatch origin, arranger, grouping, payer, quote validity, vehicle model, Big-M value, and actual-cost workflow are not selected.
- **Notes:** The architecture's fixed-plus-variable expression is future-capability, not a frozen production formula.

## Planning and revision history

### IC-REVISION-001 — Create immutable revisions for decision changes

- **Constraint family:** Planning and revision history
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §24, “Immutable revision model”; §47, “Validation layers”; §50, “PlanRevision states”
- **Architecture statement / rationale:** Optimizer, manager-modified, and manual decisions are immutable revisions; changing a decision variable creates a new revision and recomputes affected derived evidence.
- **Technology-independent implementation obligation:** Preserve revision lineage and create a new evaluated revision for every decision-variable change rather than editing prior history.
- **Required invariant:** An existing revision's decision variables and evaluation basis never change; a new revision references its predecessor/plan context and is independently validated.
- **Failure semantics:** Attempted historical mutation or stale derived-evidence reuse is rejected explicitly.
- **Audit / debugging implications:** Retain origin, lineage, changed decisions, author, time, derived recomputation, and validation outcome.
- **Testing implications:** Verify all origins, immutable prior revisions, lineage, recomputation, and no reopening of rejected/superseded revisions.
- **Security / tenant-isolation implications:** Revision creation/modification authority and all referenced plan objects remain FPO-scoped.
- **Related UNKNOWN / blocker references:** `PILOT-003`.
- **Explicit non-decisions:** Persistence immutability mechanism, revision identifier, diff representation, DTO, and UI workflow are not selected.
- **Notes:** None.

### IC-REVISION-002 — Freeze each revision's complete evaluation basis

- **Constraint family:** Planning and revision history
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §25, “Revision-specific frozen evaluation basis”; §26, “OptimizationRun scope”; §48, “Proposal freshness”; §71, “Final mental model”
- **Architecture statement / rationale:** Every evaluated revision traces to exact facts, derived evidence, policies, and numeric/validation versions used at evaluation time.
- **Technology-independent implementation obligation:** Capture a revision-specific immutable evaluation basis sufficient for replay, explanation, validation, and comparison after evidence changes.
- **Required invariant:** No evaluated revision depends on an unversioned mutable external fact; later updates do not rewrite its basis.
- **Failure semantics:** Missing or internally inconsistent basis prevents claiming reproducible evaluation/proposal validity.
- **Audit / debugging implications:** Answer which quantities, dates, mandis, forecasts, storage/route/transport evidence, policies, normalization, and validators were used.
- **Testing implications:** Verify frozen snapshot replay, changed-evidence isolation, manager-modified recomputation, and complete provenance.
- **Security / tenant-isolation implications:** Frozen evidence must belong to or be authorized for the revision's FPO.
- **Related UNKNOWN / blocker references:** `CORE-004`, `CORE-005`, `CORE-006`, `CORE-007`, `CORE-008`, `PILOT-011`.
- **Explicit non-decisions:** Snapshot storage, serialization, hashing, schema, and retention mechanism are not selected.
- **Notes:** None.

### IC-RUN-001 — Represent actual optimizer execution as a reproducible run

- **Constraint family:** Planning and revision history
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §26, “OptimizationRun scope”; §44, “Solver verification”; §57, “OptimizationRun lifecycle vs outcome”
- **Architecture statement / rationale:** Optimizer generation creates an OptimizationRun for snapshot preparation, validation, execution, output validation, failure tracking, retries, benchmarks, and reproducibility; manual-only revisions do not fabricate runs.
- **Technology-independent implementation obligation:** Track each real optimization execution independently from its eventual proposal/human outcome and retain enough evidence to reproduce and diagnose it.
- **Required invariant:** Every optimizer invocation has one run identity/lifecycle; no manual-only evaluation claims a run; rejection/communication cannot retroactively alter run outcome.
- **Failure semantics:** Input-invalid, proven infeasible, no useful allocation, technical failure, output-invalid, and proposal-created remain distinct outcomes.
- **Audit / debugging implications:** Record snapshot/basis identity, configuration, lifecycle transitions, termination, diagnostics, validation, retry lineage, and produced result/revision.
- **Testing implications:** Verify lifecycle/outcome separation, retries/idempotency, manual exclusion, failure injection, and replay.
- **Security / tenant-isolation implications:** Run data and inputs inherit the plan's FPO authorization.
- **Related UNKNOWN / blocker references:** None.
- **Explicit non-decisions:** Async mechanism, worker/queue, run-state enum names, persistence, retry values, and orchestration framework are not selected.
- **Notes:** None.

### IC-REVISION-003 — Preserve decision states without overloading operational state

- **Constraint family:** Planning and revision history
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §13, “Cancellation modeling”; §50, “PlanRevision states”; §56, “Orthogonal state model”
- **Architecture statement / rationale:** Pre-approval decision states are distinct from supersession, cancellation, execution, communication, and freshness; approved history is not edited back into proposal state.
- **Technology-independent implementation obligation:** Maintain immutable decision-state transitions separately from post-approval operational quantity/state axes.
- **Required invariant:** Rejected/superseded revisions are not reopened; approved revisions are not edited; cancellation/execution cannot overwrite historical approval.
- **Failure semantics:** Invalid transition is rejected explicitly and does not mutate history.
- **Audit / debugging implications:** Record transition actor/time/reason and preserve every independent operational axis.
- **Testing implications:** Verify allowed/forbidden transitions, superseded-versus-rejected distinction, and post-approval cancellation without decision-state mutation.
- **Security / tenant-isolation implications:** Decision transitions require authorized FPO actor/context.
- **Related UNKNOWN / blocker references:** `PILOT-003`.
- **Explicit non-decisions:** Exact enums, state-machine library, storage representation, and aggregate UI labels are not selected.
- **Notes:** Protects `RR-007`.

### IC-REVISION-004 — Replan from current truth after commitment changes

- **Constraint family:** Planning and revision history
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §51, “Revision vs replan”; §55, “Execution state”; §71, “Final mental model”
- **Architecture statement / rationale:** Pre-commit changes create revisions in one planning cycle; after approval/execution/deviation/cancellation, a new current-data planning cycle is required.
- **Technology-independent implementation obligation:** Prevent post-commit business change from being modeled as edits/new revisions under the committed plan; release remaining commitment as needed and create a new plan/run from current authoritative inventory and evidence.
- **Required invariant:** Old approved/executed history remains unchanged and the new planning cycle consumes current, not historical, availability/evidence.
- **Failure semantics:** Attempted post-commit historical edit is rejected; replan preparation failure does not alter the prior plan/reality.
- **Audit / debugging implications:** Link new cycle to triggering execution/deviation/cancellation and record current input basis.
- **Testing implications:** Verify revision before approval, replan after partial sale/cancellation, current remainder usage, and preservation of old history.
- **Security / tenant-isolation implications:** Replan scope and linked history remain in the same authorized FPO.
- **Related UNKNOWN / blocker references:** `CORE-009`, `PILOT-011`, `PILOT-012`.
- **Explicit non-decisions:** Replan trigger automation, UI flow, scheduling, API, and plan-link representation are not selected.
- **Notes:** None.

## Validation and approval

### IC-VALIDATION-001 — Keep preparation, plan, and commit validation distinct

- **Constraint family:** Validation and approval
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §27, “Optimizer boundary”; §47, “Validation layers”; §49, “Approval flow”
- **Architecture statement / rationale:** OptimizationInputValidator checks prepared input, shared PlanValidator checks hard plan validity without providers, and approval-time validation checks current authoritative commit context.
- **Technology-independent implementation obligation:** Maintain three separately invocable validation boundaries with explicit inputs/responsibilities and reuse deterministic plan validation for optimizer, modified, and manual revisions.
- **Required invariant:** PlanValidator neither proves optimality nor calls providers; approval validation does not rely solely on historical snapshot availability; all plan origins face the same hard-validity rules.
- **Failure semantics:** Input-invalid, output/plan-invalid, and approval-time conflict remain distinct and identify the responsible layer.
- **Audit / debugging implications:** Record validator/version, input basis, checks performed, exclusions, results, and approval context.
- **Testing implications:** Unit-test each validator; verify shared semantics across origins, recomputed economics, strict policy, current conflicts, and no hidden provider access.
- **Security / tenant-isolation implications:** FPO consistency and authorization are checked both in prepared input and current approval context as applicable.
- **Related UNKNOWN / blocker references:** `CORE-005`, `CORE-006`, `CORE-008`, `PILOT-003`, `PILOT-011`, `PILOT-012`.
- **Explicit non-decisions:** Validator API/types, rule engine, schema, error DTO, provider freshness collection, and database mechanism are not selected.
- **Notes:** Manager choice may be suboptimal on soft objectives while remaining hard-valid.

### IC-FRESHNESS-001 — Separate historical validity, current validity, and freshness

- **Constraint family:** Validation and approval
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §25, “Revision-specific frozen evaluation basis”; §48, “Proposal freshness”; §49, “Approval flow”
- **Architecture statement / rationale:** Historical revision validity is immutable, current commit validity and freshness are separate assessments, and evidence types do not share a universal TTL.
- **Technology-independent implementation obligation:** Assess source-specific freshness/version changes without rewriting the revision and route detected conditions through an explicit versioned approval-freshness policy.
- **Required invariant:** Historical evaluation basis remains intact; freshness observation does not itself determine warning/block action except through the active policy.
- **Failure semantics:** Unverified/unavailable/changed/expired evidence is explicit; blocking versus warning cannot be guessed, and small changes do not automatically replan.
- **Audit / debugging implications:** Record prior/current evidence versions/times, freshness observations, policy/version, materiality outcome, manager acknowledgement, and resulting action.
- **Testing implications:** Verify each evidence type, changed/unavailable/unknown states, warning/block paths after policy definition, history preservation, and no universal TTL.
- **Security / tenant-isolation implications:** Freshness evidence and acknowledgements remain associated with authorized FPO/revision.
- **Related UNKNOWN / blocker references:** `PILOT-011`, `PILOT-012`.
- **Explicit non-decisions:** TTLs, materiality thresholds, warning/block rules, acknowledgement flow, auto-refresh, and policy implementation are not selected.
- **Notes:** None.

### IC-APPROVAL-001 — Commit approval and reservations against current truth atomically

- **Constraint family:** Validation and approval
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §12, “QuantityReservation”; §47, “Validation layers”; §49, “Approval flow”; §59, “Idempotency and concurrency”
- **Architecture statement / rationale:** Approval resolves the exact immutable revision, checks authorization/idempotency/freshness, then performs a short atomic current-inventory recheck, reservation creation, and approval commit without holding locks across provider calls.
- **Technology-independent implementation obligation:** Make approval an idempotent concurrency-safe commit operation whose external/freshness preparation precedes the critical authoritative inventory transaction.
- **Required invariant:** Approval and all reservations commit together or not at all against current availability; duplicate/concurrent requests cannot double reserve.
- **Failure semantics:** Freshness/policy/input failure occurs before commitment; inventory conflict aborts commitment explicitly; communication/provider failure cannot partially commit approval.
- **Audit / debugging implications:** Record request/idempotency identity, exact revision, actor/authorization, prepared context, freshness outcome, current recheck, reservation quantities, and commit result.
- **Testing implications:** Concurrent approval, duplicate requests, stale context, inventory change, atomic rollback, and no network wait within critical transaction.
- **Security / tenant-isolation implications:** Authoritative authorization and tenant checks precede and participate in commitment.
- **Related UNKNOWN / blocker references:** `PILOT-003`, `PILOT-004`, `PILOT-011`, `PILOT-012`.
- **Explicit non-decisions:** Transaction isolation, locking, idempotency storage, database, API, and freshness-fetch mechanism are not selected.
- **Notes:** None.

## Execution and communication

### IC-COMMS-001 — Communicate only approved assignments with independent delivery state

- **Constraint family:** Execution and communication
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §52, “Communication”; §56, “Orthogonal state model”; §57, “OptimizationRun lifecycle vs outcome”
- **Architecture statement / rationale:** Approved assignments source instructions; channels do not rerun optimization; communication state is separate and failure does not undo approval/reservation or run success.
- **Technology-independent implementation obligation:** Resolve communication content from an approved immutable assignment and track each delivery attempt/result independently from planning, approval, and execution.
- **Required invariant:** No channel creates/recomputes a plan, and communication outcome cannot mutate approval/reservation/run outcome.
- **Failure semantics:** Delivery failure is explicit communication failure only; retries must not duplicate unrelated business commitment.
- **Audit / debugging implications:** Record assignment/revision, recipient/channel, content/version, attempt/idempotency identity, timestamps, status, and provider response provenance.
- **Testing implications:** Verify unapproved instructions blocked, provider failure isolation, duplicate webhook/delivery handling, and state-axis independence.
- **Security / tenant-isolation implications:** Recipient/instruction access and content derive from authorized FPO assignments.
- **Related UNKNOWN / blocker references:** `PILOT-004`.
- **Explicit non-decisions:** Channel/provider, exact status enum, message template, webhook protocol, and retry values are not selected.
- **Notes:** None.

### IC-EXECUTION-001 — Preserve planned and actual execution independently

- **Constraint family:** Execution and communication
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §3, “Fundamental architecture rule”; §53, “Planned vs actual execution”; §61, “Audit and feedback”
- **Architecture statement / rationale:** Recommendation and reality retain separate mandi/date/quantity/price values to support evaluation, adherence, overrides, and audit.
- **Technology-independent implementation obligation:** Record actual outcomes as new authoritative facts linked to, but never overwriting, planned assignments and predicted values.
- **Required invariant:** Planned values remain historically immutable after any actual sale/deviation; actual values remain independently queryable.
- **Failure semantics:** Missing actual data is not inferred from the plan, and deviation is not hidden by rewriting the plan.
- **Audit / debugging implications:** Reconstruct plan-versus-actual differences and associated actor/reason/time.
- **Testing implications:** Verify matching and deviating executions, partial/multiple sales, plan preservation, and evaluation calculations.
- **Security / tenant-isolation implications:** Planned and actual linked records share authorized lot/FPO scope.
- **Related UNKNOWN / blocker references:** `PILOT-005`, `PILOT-006`.
- **Explicit non-decisions:** Deviation analytics, UI, schema, DTO, and actual-expense capture mechanism are not selected.
- **Notes:** None.

### IC-EXECUTION-002 — Anchor ActualSale to ProduceLot with optional assignment linkage

- **Constraint family:** Execution and communication
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §14, “Off-plan ActualSale vs active reservation”; §54, “ActualSale”; §67, “Revised/rejected ideas”
- **Architecture statement / rationale:** ActualSale fundamentally belongs to ProduceLot, may optionally reference a SellingAssignment, and one assignment may have multiple sales; off-plan reality remains recordable.
- **Technology-independent implementation obligation:** Preserve lot ownership and exact actual mandi/date/quantity/price for every sale while permitting linked, partial, multiple, and off-plan execution.
- **Required invariant:** Every ActualSale references one ProduceLot; assignment linkage, if present, is valid for that lot and cannot make ActualSale dependent on assignment existence.
- **Failure semantics:** Missing assignment is not invalid by itself; invalid linkage is explicit; off-plan reservation conflict invokes unresolved reconciliation rather than arbitrary behavior.
- **Audit / debugging implications:** Record recorder, lot, optional assignment, actual values, time, and deviation/reconciliation context.
- **Testing implications:** Verify zero/many sales per assignment, off-plan sale, invalid linkage, exact quantities, and planned/actual separation.
- **Security / tenant-isolation implications:** Sale recorder authorization is unresolved; lot/assignment/sale must share FPO scope.
- **Related UNKNOWN / blocker references:** `CORE-009`, `PILOT-005`.
- **Explicit non-decisions:** Recording authority/workflow, reconciliation, schema, API, and deferred ExecutionAttempt are not selected.
- **Notes:** Protects `RR-003`.

### IC-EXECUTION-003 — Derive execution state from actual quantities and expose deviations

- **Constraint family:** Execution and communication
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §55, “Execution state”; §56, “Orthogonal state model”; §51, “Revision vs replan”
- **Architecture statement / rationale:** Execution state derives from sold versus committed target; actual overage becomes an inventory/reconciliation deviation and real changes feed a new planning cycle.
- **Technology-independent implementation obligation:** Compute execution progress from exact actual quantities and surface deviations without manual state toggles or historical plan edits.
- **Required invariant:** Derived execution status agrees with exact cumulative linked actual quantity; any overage still preserves lot inventory invariant.
- **Failure semantics:** Quantity inconsistency/deviation is explicit and cannot be normalized away by editing target/status.
- **Audit / debugging implications:** Record cumulative actuals, target, derived state, deviation quantity/reason, and replan linkage.
- **Testing implications:** Verify not-started/partial/completed derivation, multiple sales, overage, cancellation/release interactions, and replanning remainder.
- **Security / tenant-isolation implications:** Execution aggregation cannot cross assignment/lot/FPO boundaries.
- **Related UNKNOWN / blocker references:** `CORE-009`, `PILOT-005`.
- **Explicit non-decisions:** Exact status enum names, deviation reconciliation workflow, tolerance, UI, and replan trigger are not selected.
- **Notes:** None.

## Reliability, state, time, and audit

### IC-STATE-001 — Keep operational state axes orthogonal

- **Constraint family:** Reliability, state, time, and audit
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §50, “PlanRevision states”; §56, “Orthogonal state model”; §57, “OptimizationRun lifecycle vs outcome”
- **Architecture statement / rationale:** Run lifecycle/outcome/proof/termination, revision decision, freshness, communication, execution, and reservation are independent dimensions.
- **Technology-independent implementation obligation:** Represent and transition each state dimension independently, deriving aggregates where useful rather than creating giant combined mutable statuses.
- **Required invariant:** A change in one axis cannot silently change another unrelated axis; historical decisions remain immutable.
- **Failure semantics:** Failure is attributed to the responsible axis; manager rejection/delivery failure never changes solver/run success.
- **Audit / debugging implications:** Record axis-specific states, transitions, causes, times, and correlations.
- **Testing implications:** Verify valid cross-products, independent transitions, derived aggregates, and prohibited coupling.
- **Security / tenant-isolation implications:** State transitions require the authority appropriate to their axis/FPO.
- **Related UNKNOWN / blocker references:** `PILOT-003`, `PILOT-011`.
- **Explicit non-decisions:** Exact enums, state-machine framework, aggregate labels, schema, and transition API are not selected.
- **Notes:** None.

### IC-RELIABILITY-001 — Make unknown and unavailable first-class

- **Constraint family:** Reliability, state, time, and audit
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §16, “IoT / StorageAssessment”; §19, “Routing”; §21, “External provider pattern”; §28, “Technical admissibility”; §42, “OptimizationResult”; §58, “Reliability philosophy”
- **Architecture statement / rationale:** Missing/unavailable/invalid/stale facts and technical failures retain explicit semantics and never become plausible zeros, safe categories, or business infeasibility.
- **Technology-independent implementation obligation:** Define explicit absence, staleness, invalidity, unavailability, and technical-error outcomes at every evidence/computation boundary.
- **Required invariant:** No fallback value can be mistaken for observed/derived business truth.
- **Failure semantics:** Forecast unavailable ≠ zero price; route unavailable ≠ zero distance/blocked road; transport unavailable ≠ zero cost; sensor stale ≠ LOW; solver crash ≠ infeasible.
- **Audit / debugging implications:** Record origin, status/reason, timestamps, attempts, validation, and downstream exclusion/warning.
- **Testing implications:** Inject every failure/absence class and verify no fabricated domain value or semantic collapse.
- **Security / tenant-isolation implications:** Error detail must not leak other tenants' data or provider secrets.
- **Related UNKNOWN / blocker references:** `PILOT-007`, `PILOT-008`, `PILOT-011`.
- **Explicit non-decisions:** Error taxonomy names, transport/error protocol, retry values, UI presentation, and fallback policy are not selected.
- **Notes:** None.

### IC-RELIABILITY-002 — Provide idempotent, concurrency-safe external effects

- **Constraint family:** Reliability, state, time, and audit
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §11, “Inventory architecture”; §12, “QuantityReservation”; §14, “Off-plan ActualSale vs active reservation”; §49, “Approval flow”; §59, “Idempotency and concurrency”
- **Architecture statement / rationale:** Duplicate/racing MQTT, plan generation, approval, cancellation/release, ActualSale, and communication events must not repeat or corrupt authoritative effects.
- **Technology-independent implementation obligation:** Define idempotency identity and concurrency semantics for every duplicate-prone authoritative operation and commit critical inventory/approval effects atomically.
- **Required invariant:** Replaying the same logical operation does not create a second effect; concurrent conflicting effects cannot violate inventory or tenant invariants.
- **Failure semantics:** Duplicate returns/reveals the prior logical result or explicit conflict; partial/racy state is not committed.
- **Audit / debugging implications:** Record operation/event identity, attempts, deduplication result, conflict, transaction outcome, and correlation.
- **Testing implications:** Duplicate delivery, concurrent approve/sell/release, retry after ambiguous response, webhook repetition, and randomized mutation sequences.
- **Security / tenant-isolation implications:** Idempotency identity and conflict lookup cannot cross tenant scope; authorization still applies on retries.
- **Related UNKNOWN / blocker references:** `CORE-009`, `PILOT-003`, `PILOT-005`, `PILOT-008`.
- **Explicit non-decisions:** Idempotency storage, key format, locking, isolation level, database constraint, queue, and retry policy are not selected.
- **Notes:** None.

### IC-TIME-001 — Separate UTC technical time from FPO business time

- **Constraint family:** Reliability, state, time, and audit
- **Architecture classification:** `DESIGN DECISION; UNKNOWN relationship`
- **Source:** §17, “Forecasting”; §19, “Routing”; §48, “Proposal freshness”; §60, “Time”
- **Architecture statement / rationale:** Technical timestamps use UTC, each FPO has explicit local timezone for business dates/times, and naive timestamps are forbidden; operational cutoffs remain unknown.
- **Technology-independent implementation obligation:** Preserve timezone-aware instants and explicit FPO-local business-date interpretation across evidence, planning, approval, communication, and execution.
- **Required invariant:** Every technical instant is unambiguous and every business date/time resolves using the relevant FPO timezone.
- **Failure semantics:** Missing/invalid timezone or ambiguous local time is explicit invalid input/configuration; no cutoff is inferred.
- **Audit / debugging implications:** Record UTC instant, relevant timezone/version/configuration, and derived local business date/time.
- **Testing implications:** Timezone conversion, date boundaries, offset changes, ambiguous/nonexistent local times, and cutoff behavior only after validation.
- **Security / tenant-isolation implications:** FPO timezone configuration is tenant-scoped and authorized.
- **Related UNKNOWN / blocker references:** `PILOT-012`.
- **Explicit non-decisions:** Time library, storage type, timezone identifier representation, cutoff values, and scheduling mechanism are not selected.
- **Notes:** None.

### IC-AUDIT-001 — Separate immutable audit facts from decision feedback

- **Constraint family:** Reliability, state, time, and audit
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §3, “Fundamental architecture rule”; §25, “Revision-specific frozen evaluation basis”; §61, “Audit and feedback”; §71, “Final mental model”
- **Architecture statement / rationale:** Audit answers who/what/when, feedback answers why; meaningful decisions/deviations capture structured reason plus note, and feedback does not directly retrain models.
- **Technology-independent implementation obligation:** Preserve reconstructable audit history independently from structured explanatory feedback and require human investigation/validated versioning before feedback changes models/policies.
- **Required invariant:** Audit history is not overwritten by feedback/correction; feedback alone cannot mutate production model/policy truth.
- **Failure semantics:** Missing feedback does not erase the event; failed feedback/model analytics cannot rewrite history or trigger implicit retraining.
- **Audit / debugging implications:** The system must answer the §71 fact/version/optimizer/validation/manager/reservation/communication/reality questions for important results.
- **Testing implications:** Verify reconstruction, correction lineage, reason capture, audit/feedback separation, and no automatic retraining.
- **Security / tenant-isolation implications:** Audit/feedback access, actor identity, and notes remain appropriately FPO-scoped and protected.
- **Related UNKNOWN / blocker references:** `PILOT-003`, `PILOT-005`, `PILOT-006`.
- **Explicit non-decisions:** Event sourcing, audit store/schema, reason codes, retention, analytics, and retraining process are not selected.
- **Notes:** No event-sourcing platform is required for V1.

## Verification

### IC-VERIFY-001 — Verify contracts at unit, provider, integration, end-to-end, and property levels

- **Constraint family:** Verification
- **Architecture classification:** `DESIGN DECISION`
- **Source:** §44, “Solver verification”; §46, “Numerical architecture”; §47, “Validation layers”; §62, “Testing strategy”
- **Architecture statement / rationale:** Correctness requires layered tests, provider contracts, approval/concurrency integration, realistic end-to-end execution, randomized inventory invariants, replay/differential/failure tests, and audit reconstruction.
- **Technology-independent implementation obligation:** Attach verification at the narrowest responsible boundary and maintain cross-boundary scenarios for invariant, concurrency, failure, and historical behavior.
- **Required invariant:** No frozen implementation contract is considered complete without named tests for its invariants and failure semantics.
- **Failure semantics:** Test/benchmark inability or failure blocks claiming the affected contract/choice verified; it cannot be relabelled as business infeasibility.
- **Audit / debugging implications:** Retain test inputs, environment/config where material, seeds, expected contracts, results, and deviations.
- **Testing implications:** This constraint defines the required test layers without selecting frameworks; include exact arithmetic, tenant isolation, provider failures, approval races, replay, and full operational flow.
- **Security / tenant-isolation implications:** Security and cross-FPO isolation require direct negative/property/integration coverage.
- **Related UNKNOWN / blocker references:** `CORE-009`; final reconciliation integration tests wait for its resolution.
- **Explicit non-decisions:** Test framework, CI platform, coverage target, fixture technology, benchmark thresholds, and environment are not selected.
- **Notes:** No application test suite exists during blueprint extraction because product code is absent.

## UNKNOWN validation gates

### IC-UNKNOWN-001 — Gate affected production contracts on the nine core blockers

- **Constraint family:** UNKNOWN validation gates
- **Architecture classification:** `UNKNOWN relationship`
- **Source:** §14, “Off-plan ActualSale vs active reservation”; §20, “TransportCostService”; §31, “MarketLoadPolicy”; §32, “EconomicPolicy”; §63, “Real-FPO unknowns — priority classification”; §68, “Things that must NOT be invented in implementation”
- **Architecture statement / rationale:** Nine real-FPO questions block freezing their affected production behavior but do not authorize guessed defaults.
- **Technology-independent implementation obligation:** Keep affected boundaries configurable/unimplemented as appropriate until explicit approved decisions backed by required evidence resolve each core blocker.
- **Required invariant:** No production contract encodes an answer for `CORE-001` through `CORE-009` while its register entry is `OPEN`.
- **Failure semantics:** Missing decision/evidence is an explicit governance/readiness blocker, not a runtime default.
- **Audit / debugging implications:** Link each affected future contract/task to blocker status, evidence, decision, and resolution history.
- **Testing implications:** Add readiness checks that fail if an unresolved blocker is presented as production behavior; later tests must cite its approved resolution.
- **Security / tenant-isolation implications:** Authority-related outcomes remain blocked where applicable; tenant isolation itself is not optional.
- **Related UNKNOWN / blocker references:** `CORE-001`, `CORE-002`, `CORE-003`, `CORE-004`, `CORE-005`, `CORE-006`, `CORE-007`, `CORE-008`, `CORE-009`.
- **Explicit non-decisions:** Custody/origin, execution path, transport arranger/pricing, economic rule, increment, quality effect, market-load production semantics, and off-plan reconciliation are not selected.
- **Notes:** Number resolved in this task: zero.

### IC-UNKNOWN-002 — Gate pilot readiness on the twelve pre-pilot validations

- **Constraint family:** UNKNOWN validation gates
- **Architecture classification:** `UNKNOWN relationship`
- **Source:** §16, “IoT / StorageAssessment”; §34, “Allocation fairness”; §37, “Storage objective”; §48, “Proposal freshness”; §60, “Time”; §64, “Real-FPO unknowns — must validate before pilot”
- **Architecture statement / rationale:** Twelve operational/scientific/workflow questions require real validation before pilot and cannot be inferred from software design.
- **Technology-independent implementation obligation:** Track pilot readiness against explicit validated evidence/decisions for every pre-pilot entry without resolving them in the constraint catalogue.
- **Required invariant:** Pilot readiness cannot be asserted while any required `PILOT-001` through `PILOT-012` entry remains `OPEN`, unless architecture/governance is explicitly updated from authoritative evidence.
- **Failure semantics:** Insufficient evidence remains an open validation gate, not a plausible workflow/default threshold.
- **Audit / debugging implications:** Retain evidence, decision authority, applicability, limitations, approval, and status transition for each validation.
- **Testing implications:** Readiness audit must enumerate all 12; future workflow/scientific tests cite approved evidence and contracts.
- **Security / tenant-isolation implications:** Role/authority validations cannot weaken server-side isolation.
- **Related UNKNOWN / blocker references:** `PILOT-001`, `PILOT-002`, `PILOT-003`, `PILOT-004`, `PILOT-005`, `PILOT-006`, `PILOT-007`, `PILOT-008`, `PILOT-009`, `PILOT-010`, `PILOT-011`, `PILOT-012`.
- **Explicit non-decisions:** Registration/verification workflow, roles, farmer acceptance, sale/expense recording, thresholds, IoT settings, policy acceptance/trade-offs, freshness rules, and cutoffs are not selected.
- **Notes:** Number resolved in this task: zero.
