# L3 slice `plan/v1` — baseline forecast with ranges, sensor link, and the first plan proposal

26 October 2026 • **Status: FROZEN** (contract version `plan/v1`) on the owner's approval of slice 2 of the product flow ("Yes, start slice 2", 26 Oct; scope in [PRODUCT_FLOW_STATUS.md](../PRODUCT_FLOW_STATUS.md): S2.1, S2.2, S2.4 link, S2.6 to S2.9). Schemas and examples: `kisaniti_implementation/contracts/plan/v1/`. Builds on `shared/v1`, `org/v1`, `market/v1`, `devices/v1` and `allocation/v1` (unchanged). Recorded as CHAT-44. **Values marked DEMO policy are chosen settings, not evidence about any real FPO; they are named, versioned and configuration, not contract.**

## 0. What this slice is and is not

It produces a **proposal**: a system decision that the manager will later review. It is not an approval, not a reservation, not a message. Stock is not reduced by a proposal. Approval, reservation and rejection are the next slice (stage 3) and will be separate human-decision objects that point at an immutable proposal; nothing here changes a proposal after it is stored.

**What the forecast is.** A baseline: the naive and the 5-day moving-average rules of the draft pre-registered protocol ([PREREGISTERED_PROTOCOL.md](../../../kisaniti_research_paper/layers/L3a_forecasting/PREREGISTERED_PROTOCOL.md) §5), with empirical ranges. The protocol is still a DRAFT and no model has been compared on held-out data. Therefore every forecast says so on its face (`label`, `signal_verdict = NOT_ASSESSED`), the plan uses the naive rule (protocol §9: "naive, and say the baseline is enough"), and no sentence of the product may call a forecast "good", "accurate" or "calibrated". The rule for the words "no reliable signal" (CHAT-22: when a range is too wide) is **UNKNOWN until the interval experiment EXP-04 is run**; this slice does not invent a width threshold.

## 1. Policies (DEMO / ENGINEERING LEAN, versioned names)

| Name | Rule | Class |
|---|---|---|
| `INDEX_DATED_ONLY_DEMO_V1` | Variety index as in `layers/L2_cleaning/build_index.py`: for one variety and day, the median of the modal prices of the mandis that reported the variety that day, only if at least 3 mandis reported; rows without their own date (`date_inferred`) are not used; a mandi-day with two or more rows for the variety is excluded, never averaged; rows with no modal price are skipped; sub-yards are **not** merged (O9 open); only the varieties `1121` and `Basmati 1509`. Computed exactly (decimals); a median of an even count is the exact mean of the two middle values (up to three decimals) | DEMO until the hand check (B.2) decides between `dated_only` and `with_blank`; this is the conservative variant |
| `BASELINE_FORECAST_V1` | `NAIVE`: the last index value at the origin. `MOVING_AVERAGE_5`: the mean of the last 5 index days of the same season up to the origin. Each point forecast is rounded half-even to the paisa at the end; errors and range half-widths are computed from the exact unrounded values, and a range bound is the exact point ± half-width rounded half-even to the paisa. The origin is the newest index day of the newest season. A target is the index value on exactly origin + h calendar days. Horizons h ∈ {1, 7, 14} (the protocol's primary horizons). A forecast is available only after 10 index days of the current season exist (protocol §3) | ENGINEERING LEAN, from the protocol draft |
| `RANGE_ABS_ERROR_CONFORMAL_V1` | Ranges are symmetric around the point forecast. For method M and horizon h, collect every past pair (origin o, target o+h), both index days of the same season, with target date ≤ the forecast origin; error = \|actual − M's forecast at o\|; with n pairs sorted ascending, the half-width for coverage c is the ⌈(n+1)c⌉-th smallest error (split-conformal form); coverage 80 % and 95 %. Fewer than **40** pairs, or ⌈(n+1)c⌉ > n: no range (`TOO_FEW_PAIRS`). Taken from protocol §6 (no coverage claim from fewer than 40 pairs). Differences from the protocol: it uses all earlier seasons, not the validation season; its calibration is **not verified** (`calibration = NOT_VERIFIED`) | ENGINEERING LEAN |
| `PLAN_USES_NAIVE_DEMO_V1` | The plan uses the `NAIVE` forecast | DEMO until the protocol is run |
| `BASIS_RECENT_MEDIAN_DEMO_V1` | A mandi's price level relative to the index: over the 30 calendar days ending at the origin, the median of (that mandi's modal price for the variety − the index that day), on days where both exist and the mandi has exactly one row; at least **5** such days, else the mandi is not eligible (`BASIS_TOO_FEW_DAYS`). The price of mandi m at horizon h is the forecast point + the basis | DEMO |
| `TRANSPORT_AS_ENTERED_V1` | The cost of reaching a mandi is **entered by the manager** per mandi and per quintal (two decimals, 0.00 allowed). The system has no transport model (UNKNOWN); an entered 0.00 is shown as such in the explanation | human input |
| `WAITING_COST_NOT_MODELLED_V1` | Waiting cost, storage loss and interest are **not** in the cost of a choice (hold-or-sell, S2.5, is a later step). The explanation says so every time | DEMO |
| `STORAGE_FROM_LINKED_SENSOR_V1` | A lot's storage category is that of the latest stored assessment of the sensor linked to the lot, if the assessment is at most 2 hours old and the category is HIGH, MEDIUM or LOW; otherwise UNKNOWN with a reason (`NO_SENSOR_LINK`, `ASSESSMENT_OLD`, or the assessment's own unknown reason). UNKNOWN gets no priority and no invented rank | DEMO (2-hour age) |
| `PLAN_LOTS_V1` | A plan covers the lots whose `source_variety` equals the requested variety exactly and whose available stock (on hand − reserved, at the time of the request) is at least one increment; the quantity planned is the available stock. Others are listed as excluded with a reason | DEMO |
| `PLAN_INCREMENT_DEMO_V1` | One increment size for all lots, entered by the manager (default shown 100.000 kg). A remainder below one increment stays unallocated and is reported (PARTIAL mode) | DEMO (real increments UNKNOWN) |
| Solver | `highs`, hierarchy and time limit from configuration (`KN_PLAN_TIME_LIMIT_S`, default 30 s; the real limit is E.0, open). Allocation mode `PARTIAL`, market load `DISABLED` (D19). A result that is not `PROVEN_OPTIMAL` is stored but **not proposed** (G09 is open) | DEMO |

## 2. Objects and endpoints (all under `/api/v1`)

| Object | Schema | Notes |
|---|---|---|
| `ForecastSetV1` | `forecast_set.schema.json` | derived, computed on request from the newest non-rejected price dataset; never stored by itself (a plan stores its own snapshot) |
| `SensorLinkV1` | `sensor_link.schema.json` | append-only; the current link of a lot is its latest row; `device_id` null unlinks |
| `SensorLinkRequestV1` | `sensor_link_request.schema.json` | |
| `PlanRequestV1` | `plan_request.schema.json` | |
| `PlanProposalV1` | `plan_proposal.schema.json` | immutable once stored |
| `PlanSummaryV1` | `plan_summary.schema.json` | list entry |

| Method and path | Needs | Does | Success | Errors |
|---|---|---|---|---|
| `GET /forecasts?variety=…` | `data.read` | the forecast set for `1121` or `Basmati 1509`, the candidate mandis with their basis, the last 30 index points | 200 `ForecastSetV1` | 422 `VARIETY_NOT_FORECAST`, 404 `NO_PRICE_DATA`, 422 `INSUFFICIENT_HISTORY` (fewer than 10 index days in the season) |
| `GET /sensor-links/current` | `data.read` | the current link of every lot of the caller's FPO that has one | 200 list of `SensorLinkV1` | |
| `GET /lots/{id}/sensor-links` | `data.read` | the history of one lot | 200 list | 404 for another FPO's lot |
| `POST /lots/{id}/sensor-links` | `data.write`, CSRF | links the lot to a device of the same FPO, or unlinks (`device_id` null) | 201 `SensorLinkV1` | 404 (lot or device of another FPO, same as missing), 422 |
| `POST /plans` | `data.write`, CSRF, optional `Idempotency-Key` | freezes the inputs, runs the optimiser, validates, stores the proposal | 201 `PlanProposalV1` (also when `status` is `NOT_PROPOSED`) | 422 issue codes below (also for an unknown mandi: mandis are global), 409 `IDEMPOTENCY_KEY_REUSED` |
| `GET /plans` | `data.read` | the FPO's plans, newest first | 200 list of `PlanSummaryV1` | |
| `GET /plans/{id}` | `data.read` | one plan | 200 `PlanProposalV1` | 404 for another FPO's plan (audited as a cross-tenant reference) |

Request issue codes (422 `VALIDATION_FAILED`): `VARIETY_NOT_FORECAST`, `HORIZON_NOT_ALLOWED`, `NO_MANDI_SELECTED`, `MANDI_NOT_ELIGIBLE`, `TRANSPORT_INVALID`, `INCREMENT_INVALID`, `NO_ELIGIBLE_LOTS`, `NO_PRICE_DATA`, `INSUFFICIENT_HISTORY`, `PRICE_NOT_POSITIVE` (point + basis − 0 ≤ 0).

## 3. The proposal document (what is frozen inside it)

- `status`: `PROPOSED` (optimality proven, a solution found with something allocated) or `NOT_PROPOSED` with `not_proposed_reasons` from `NO_SOLUTION`, `NOTHING_ALLOCATED`, `OPTIMALITY_NOT_PROVEN`, `TECHNICAL_FAILURE`. A `NOT_PROPOSED` plan is stored for audit and cannot be approved later.
- `request` (as entered), `dataset` (id, name, validation status, manifest hash), `forecast` (the `NAIVE` and `MOVING_AVERAGE_5` series at the plan's horizons, as shown), `mandis` (each selected mandi: basis, days used, transport as entered, eligible or the reason it was not), `lots` (each lot of the FPO of that variety: planned quantity or the exclusion reason; storage category with its reason and the device), `allocation_input` (the exact `allocation_input/v1` document the optimiser received) and its SHA-256, `allocation_result` (the exact `allocation_result/v1` document), `validation` (the independent `plan_validation/v1` report of the same plan), `policy_versions`, `origin` (SYNTHETIC if any member or sensor involved is synthetic) and `provenance`.
- `warnings`: a list of codes that need attention before a manager acts: `PRICE_DATASET_UNCHECKED`, `PRICE_DATA_OLDER_THAN_TODAY` (params: days), `RANGE_NOT_AVAILABLE`, `STORAGE_ALL_UNKNOWN`, `NOT_ALL_STOCK_ALLOCATED`, `SYNTHETIC_DEMO_DATA`, `PRICE_SAME_ON_ALL_DATES`.
- `explanation`: a list of `{code, params}`; **the backend never writes prose** (CHAT-36); the website translates codes. Codes: `FORECAST_METHOD`, `FORECAST_NOT_VALIDATED`, `PRICE_DATASET_STATUS`, `PRICE_DATA_AGE`, `MANDI_BASIS`, `MANDI_EXCLUDED`, `TRANSPORT_AS_ENTERED`, `WAITING_COST_NOT_MODELLED`, `LOT_EXCLUDED`, `STORAGE_PRIORITY`, `STORAGE_UNKNOWN`, `INCREMENT_REMAINDER`, `FAIRNESS_RULE`, `FARMER_FULFILMENT`, `DATE_BY_TIE_BREAK`, `OUTCOME`. Parameters are strings, quantities and money as the exact wire strings.
- The proposal is a function of its frozen inputs: the same `allocation_input` yields the same optimum business tuple; the tie-break limit of `allocation/v1` §3 applies.

## 4. Rules (binding)

1. **Separation.** Planned prices are forecasts and say so; nothing in a proposal is an actual price, an actual sale or a reservation.
2. **Tenant scope.** The FPO is the principal's. A plan reads and freezes only that FPO's lots and members. Another FPO's plan, lot or device answers 404. Row-level security protects `plans` and `sensor_links`.
3. **Exactness.** Quantities in grams, rates in 1/10 000 rupee; the forecast and basis are exact decimals; rounding happens only where a policy above says so (the moving average, to the paisa, half-even).
4. **No silent substitution.** A missing price, basis, assessment or link is reported as such; nothing is filled with a default except a value the manager entered.
5. **Explicit failure.** The optimiser's status is stored; a result that is not proven optimal is not proposed; a technical failure is stored as `NOT_PROPOSED`, not hidden.
6. **Immutability.** `plans` rows cannot be updated or deleted (trigger), also not by the owner role.
7. **Synthetic data.** A plan whose members are synthetic is `origin = SYNTHETIC`, `mode = DEMO`; `allocation/v1` refuses `LIVE` with synthetic input.

## 5. Not in this slice (deliberate)

The model comparison and promotion rule (C.0, S2.3), hold-or-sell and waiting cost (S2.5), seasonal naive (the protocol's reading of "same calendar day" is ambiguous; settled with C.2), any transport model or route provider, mandi capacities (market load is `DISABLED`), editing a proposal, approval, reservation, rejection and release (stage 3), messages (stage 4), a live price source (S1.7).

## 6. Test obligations

Contract examples (valid pass, invalid fail); the index and baseline code against hand-computed cases and, for the real data, against `layers/L2_cleaning/build_index.py`'s own output; ranges against a brute-force count; two FPOs: no plan, lot, link or device crosses; idempotent replay and key reuse; a plan equals an independent recomputation (oracle check on a small plan, `plan_validation/v1` valid); `NOT_PROPOSED` when optimality is not proven; immutability of `plans`; the explanation contains only listed codes; no clock-dependent value inside the frozen input other than the stated dates.
