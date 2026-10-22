# Contract spec V1 — L3d allocation and fairness (W01 slice, milestone M7)

22 October 2026 • **Status: FROZEN** (`allocation/v1`) under CHAT-28. Decision W01-L3D in the [decision register](../blueprint/01_DECISION_REGISTER.md). Builds on [shared/v1](CONTRACT_SPEC_V1_shared.md). Schemas and examples: `kisaniti_implementation/contracts/allocation/v1/`. Reference code: `kisaniti_core.allocation` (`model`, `objective`, `milp`, `backends`, `oracle`, `greedy`, `validator`, `result`).

This slice fixes the **mathematical interface** of the optimizer (architecture §32–§47). It does not settle the real FPO's economic guardrail, planning increment, calendar or fairness acceptance; those stay DEMO policies with the version names below.

## 1. Documents

| Contract | Producer → consumer | Holds |
|---|---|---|
| `allocation_input/v1` | Planning snapshot → optimizer, oracle, validator | One FPO, one commodity: ordered dates, mandis, farmers, lots, admissible choices with exact prices and costs, mandi-day capacities, mode and policy versions |
| `allocation_result/v1` | Optimizer / heuristic → Layer 4 proposal builder | Status (solution, allocation outcome, optimality, termination, certified stages), sparse assignments, per-lot and per-farmer totals, exact fulfilment, storage outcome, money totals, tie-break score, embedded validation report. **Not** a proposal, approval, reservation or sale |
| `plan_validation/v1` | PlanValidator → everyone | Hard validity issues, recomputed money and quantity, mandi-day limit violations |

## 2. Input rules (OptimizationInputValidator)

1. IDs follow shared/v1. Every lot owner must be a listed farmer, and every choice must refer to a listed lot, mandi and planning date. Breaking this is `TENANT_SCOPE_VIOLATION` or `VALIDATION_FAILED`. All issues are reported together.
2. `dates` are unique and increasing. They are the whole horizon. The calendar policy that produces them is outside this slice.
3. Each lot has `quantity_kg > 0` and `increment_kg > 0` (the planning increment Δ_l). A plan sends whole increments only. In `FULL` mode, quantity must equal a whole number of increments. In `PARTIAL` mode, any remainder below one increment simply stays unallocated and is reported.
4. A choice carries `price_inr_per_kg` and `cost_inr_per_kg`. The transport model, waiting cost and route provider resolve into this one exact per-kg cost before the optimizer runs. Net per kg = price − cost. V1 uses linear costs only; fixed trip charges and shared trucks are future work (handoff §2 transport warning).
5. `market_load_policy`: `DISABLED` (demo default, D19), or `STRICT`, in which case every (mandi, date) used by a choice **must** have a `capacity_kg`. A missing capacity is an error, never "unlimited".
6. `storage` is HIGH, MEDIUM, LOW or UNKNOWN, taken from the frozen StorageAssessment. LOW and UNKNOWN get no priority; UNKNOWN is never given an invented rank.
7. `mode = LIVE` with `origin = SYNTHETIC` is refused (`SYNTHETIC_INPUT_IN_LIVE`).
8. `policy_versions` must equal the versions the core implements: `LEXIMIN_FULFILMENT_V1`, `STORAGE_PRIORITY_V1`, `TIE_BREAK_V1`, `NONE_DEMO_V1`. Any other value is refused.

## 3. Objective (frozen semantics)

Let y_e be the integer number of increments on admissible choice e = (lot, mandi, date), and x_e = Δ_l·y_e kilograms.

> **MILP formulation (KisanNiti V1)**
>
> Constraints: Σ_e∈l y_e ≤ ⌊Q_l/Δ_l⌋ (PARTIAL) or = Q_l/Δ_l (FULL); Σ_{e at (m,d)} Δ_l y_e ≤ C_md (STRICT); y_e ∈ ℤ≥0.
> Farmer fulfilment r_f = S_f / P_f, with S_f = Σ_{l owned by f} Σ_e x_e and P_f = Σ_{l owned by f} Q_l (all of a farmer's lots together; farmers with P_f = 0 are excluded).
>
> Solved in this order, each optimum kept as a constraint for the next:
> 1. **Allocation:** max Σ x_e (skipped in FULL mode).
> 2. **Leximin:** for k = 1…|F|, max L_k = k·θ_k − Σ_f w_fk subject to w_fk ≥ θ_k − r_f, w_fk ≥ 0. Here L_k is the sum of the k smallest r_f.
> 3. **Storage:** max HIGH total, then HIGH quantity on each date (earliest first); then the same for MEDIUM.
> 4. **Expected net return:** max Σ x_e (price_e − cost_e).
> 5. **Tie-break (TIE_BREAK_V1):** min Σ_e rank(e)·y_e, with rank = position in the order (lot ID, date, mandi ID).

**How the optimum is preserved.** After stage k, the k-th value v_k = L_k* − L_{k−1}* is computed exactly from the integer plan. The preserve constraint fixes θ at v_k: Σ_f (v_k − r_f)⁺ ≤ k·v_k − L_k*. This holds for exactly the plans that keep the optimal prefix L_1*…L_k*. Equal consecutive values give the same row, so only one row per distinct value is added. When v_k = 1, every later value is also 1, and the remaining stages collapse into Σ_f r_f ≥ L_|F|*. Integer-valued stages (1, 3, 4, 5) are preserved exactly.

**Two exact methods for stages 1–2 (same semantics).**
- *Flow water-filling* (`leximin_flow.py`) is used automatically when every lot has the same increment and the mode is PARTIAL. It repeatedly gives one more increment to the farmer first in the order (smallest current ratio, then smaller P_f, then farmer ID), along an augmenting path that may reroute other farmers but never reduces them.
  - With one common increment, the reachable farmer totals form an integral polymatroid.
  - Leximin equals maximising Σ_f −exp(−M·r_f) for large M, a separable concave objective.
  - Greedy is optimal for separable concave maximisation over polymatroids (Federgruen & Groenevelt 1986; Megiddo 1974 for lexicographically optimal flows).

  Its exact sorted vector is then written into the MILP as the preserve rows above, and stages 3–5 are solved by the MILP.
- *Staged MILP* (Ogryczak) is used in every other case, or when `leximin_method="milp"` is passed.

Both methods agree with each other and with the oracle in the tests. The MILP leximin can be slow even for about 8 farmers, because proving integer max-min values needs a near-zero gap. That is why the flow method is the default.

**Numerics.** Fairness truth is always the exact fraction S_f/P_f. After every stage the core checks that no earlier leximin value was lost; if one was, the result is `TECHNICAL_FAILURE`. HiGHS and SCIP run with a relative MIP gap of 0 and a 10⁻⁹ tolerance on fairness rows. CP-SAT runs in exact integers. When the common denominator of the fulfilment ratios needs coefficients above 2⁴⁰, CP-SAT reports `NOT_APPLICABLE` rather than rounding.

**Tie-break limit (recorded).** TIE_BREAK_V1 fixes the tie-break *score*. Two plans with the same business tuple and the same score can both exist; then the plan depends on solver, version and seed. Business metrics are identical either way. A truly lexicographic tie-break needs one solve per coordinate, which the architecture (§40) says not to do without benchmark evidence.

## 4. Result status (architecture §42)

`solution` FOUND/NONE. `allocation_outcome` FULL/PARTIAL/NONE. `optimality` PROVEN_OPTIMAL / FEASIBLE_NOT_PROVEN / NONE (or NOT_APPLICABLE_HEURISTIC for the greedy heuristic). `termination` COMPLETED / PROVEN_INFEASIBLE / TIME_LIMIT / NOT_APPLICABLE / TECHNICAL_FAILURE. `certified_stages` lists the stages proven optimal, so a time-limited run still says which prefix of the hierarchy holds. Whether a FEASIBLE_NOT_PROVEN result may become a proposal remains a Layer 4 policy question (G09); it is still OPEN.

## 5. PlanValidator (architecture §47)

The validator is independent: it imports neither the optimizer nor the oracle. It is used for optimizer, edited and manual plans. It checks:

- known lots only;
- admissible (lot, mandi, date) choices;
- positive quantities in whole increments;
- no duplicate assignment;
- no lot over-allocated, and FULL mode fully allocated;
- STRICT mandi-day limits.

It recomputes gross, cost and net money with the shared/v1 rule (per line, half-even to paise). Given a claimed result, it also checks the claimed totals and fulfilment fractions. It does **not** judge optimality or fairness: a manager's hard-valid plan may be less fair than the optimum.

## 6. Verification required (T30, T31)

- **T30:** the brute-force oracle enumerates every integer plan of tiny inputs. Each exact back end must match the oracle's complete objective tuple, not just the net return. FULL-mode inputs the oracle proves infeasible must return PROVEN_INFEASIBLE. Status: 162 random cases × 3 solvers pass in `core/tests/test_allocation.py`.
- **T31:** fulfilment is per farmer across all lots. The two-farmer example (100 kg and 300 kg, 200 kg capacity) gives 50 kg + 150 kg (both 50%). A discrete case where no single farmer can be fixed at the max-min value still reaches the leximin vector.

## 7. Open (not frozen here)

- EconomicPolicy (only `NONE_DEMO_V1`: every admissible choice is allowed, and negative net is not forbidden).
- Real increments and calendars.
- Fixed and shared transport charges.
- Proposal eligibility of time-limited results.
- Real-FPO acceptance of leximin and of putting storage before net return (architecture §37).
- A faster equivalent leximin method for large plans (OI-14).
