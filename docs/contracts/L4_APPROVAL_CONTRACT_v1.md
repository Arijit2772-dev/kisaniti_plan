# L4 slice `approval/v1` — revisions, the manager's decision, and the exact stock reservation

26 October 2026 • **Status: FROZEN** (contract version `approval/v1`) on the owner's approval of slice 3 ("Yes, start slice 3", 26 Oct; scope in [PRODUCT_FLOW_STATUS.md](../PRODUCT_FLOW_STATUS.md): S3.1 to S3.7). Schemas and examples: `kisaniti_implementation/contracts/approval/v1/`. Builds on `shared/v1`, `org/v1` (the stock ledger), `plan/v1` and `allocation/v1`, all unchanged. Recorded as CHAT-45. Architecture sources: handoff §12, §13, §47 to §51, §59 ([`QuantityReservation`](../architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md), approval flow, revision states, idempotency). Values marked DEMO are chosen settings, not evidence about any real FPO.

## 0. What this slice is

The system's proposal (plan/v1) is **not** a decision. Here a **person** reviews it, may change it into a new immutable revision (the independent validator must accept the change), and then either **approves** (which reserves the exact stock in the same database transaction) or **rejects**. Later a person may **release** what is still reserved. Three kinds of fact stay separate: the system's proposal (never edited), the human decisions (append-only rows with actor and time), and the stock reality (the ledger). Approval does not message anyone, sell anything or record any sale (slices 4 and 5).

## 1. Rules and DEMO policies

| Name | Rule | Class |
|---|---|---|
| Revision numbers | Revision **1** is the system proposal itself (the stored plan, not copied). Each manager edit creates revision **n + 1** (`MANAGER_EDIT`, id `rev_…`), an immutable document with its own `allocation_result/v1` and `plan_validation/v1`. The newest revision is current; older ones are `superseded` (derived, not stored) | DESIGN |
| `EDIT_WITHIN_FROZEN_INPUT_V1` | An edit supplies the full list of assignments. It may only use the (lot, mandi, date) choices of the plan's frozen `allocation_input`, quantities in whole increments, no lot over its planned quantity; the shared independent `PlanValidator` must report no issue. It may be less fair or lower in return than the optimum (architecture §47: the validator does not judge optimality). No new mandi, date, lot or price can be introduced by an edit; that needs a new plan | DESIGN |
| `APPROVAL_NEEDS_ACKNOWLEDGEMENT_V1` | To approve, the manager sends the codes of every warning of the plan and of the approval context ([§3](#3-approval-context)); anything missing is refused (`WARNINGS_NOT_ACKNOWLEDGED`). This is the explicit "manager is informed before commitment" of §48; no freshness threshold is invented (they are UNKNOWN) | DESIGN |
| One decision per plan | A plan has at most one decision, `APPROVED` or `REJECTED`, taken on the **current** revision. A decided plan is never edited, re-decided or reopened; changes after approval are a release followed by a new plan (§51) | DESIGN |
| Reservation | Approval writes one `RESERVED` ledger event per lot (the sum of that lot's assignments in the approved revision) in the same transaction as the decision. The database trigger of `org/v1` refuses any event that would make available stock negative | DESIGN |
| Release | A person with approval rights may release **all** the remaining reservation of an approved plan (one `RESERVATION_RELEASED` event per lot with a remainder). Releasing part of a plan (one assignment, one lot) is not in this slice. Executed sales are never undone (none exist yet) | DESIGN |
| Reason codes | Rejection: `PRICE_CONCERN`, `QUANTITY_CONCERN`, `MARKET_CONCERN`, `STORAGE_CONCERN`, `OTHER`. Release: `PLAN_CANCELLED`, `STOCK_NEEDED_ELSEWHERE`, `MARKET_CHANGED`, `OTHER`. No free text is stored (privacy and audit rules of `access/v1`) | DEMO (the real list is UNKNOWN) |
| Authority | Approve, reject and release need the capability `plans.approve` (role `FPO_ADMIN` in the demo fixture). Editing a revision needs `data.write`. Who may approve in a real FPO is UNKNOWN (U-roles) | DEMO |

## 2. Objects and endpoints (all under `/api/v1`)

| Object | Schema | Notes |
|---|---|---|
| `RevisionV1` | `revision.schema.json` | revision 1 has `id` null |
| `RevisionRequestV1` | `revision_request.schema.json` | `base_revision_no` and the full list of assignments |
| `ApprovalContextV1` | `approval_context.schema.json` | derived on request |
| `ApproveRequestV1`, `RejectRequestV1`, `ReleaseRequestV1` | `approve_request`, `reject_request`, `release_request` | |
| `PlanDecisionV1` | `decision.schema.json` | the stored human decision |
| `PlanStateV1` | `plan_state.schema.json` | derived: status, revisions, decision, reservation |

| Method and path | Needs | Does | Success | Errors |
|---|---|---|---|---|
| `GET /plan-states` | `data.read` | the state of each of the FPO's plans, newest first | 200 list of `PlanStateV1` | |
| `GET /plans/{id}/state` | `data.read` | one plan's state | 200 `PlanStateV1` | 404 (another FPO's plan too, audited) |
| `GET /plans/{id}/revisions` | `data.read` | every revision, oldest first, with `changes` against its base | 200 list of `RevisionV1` | 404 |
| `POST /plans/{id}/revisions` | `data.write`, CSRF, optional `Idempotency-Key` | validates the edit and stores revision n + 1 | 201 `RevisionV1` | 404; 409 `STALE_REVISION`, `ALREADY_DECIDED`, `PLAN_NOT_PROPOSED`; 422 `PLAN_INVALID` (the validator's issues in `issues`), `NO_CHANGE` |
| `GET /plans/{id}/approval-context` | `data.read` | what an approval would reserve and the warnings to acknowledge | 200 `ApprovalContextV1` | 404 |
| `POST /plans/{id}/approve` | `plans.approve`, CSRF, optional `Idempotency-Key` | decision and reservation, atomically | 201 `PlanDecisionV1` | 403; 404; 409 `STALE_REVISION`, `ALREADY_DECIDED`, `PLAN_NOT_PROPOSED`, `INSUFFICIENT_AVAILABLE`, `IDEMPOTENCY_KEY_REUSED`; 422 `WARNINGS_NOT_ACKNOWLEDGED`, `NOTHING_TO_RESERVE` |
| `POST /plans/{id}/reject` | `plans.approve`, CSRF, optional `Idempotency-Key` | rejection with a reason code | 201 `PlanDecisionV1` | 403; 404; 409 `STALE_REVISION`, `ALREADY_DECIDED`, `PLAN_NOT_PROPOSED` |
| `POST /plans/{id}/release` | `plans.approve`, CSRF, optional `Idempotency-Key` | releases the remaining reservation | 201 `PlanStateV1` | 403; 404; 409 `NOT_APPROVED`, `NOTHING_TO_RELEASE` |

A replay of a request with the same `Idempotency-Key` and the same body returns the stored answer (same status code 201) and changes nothing; the same key with another body is `409 IDEMPOTENCY_KEY_REUSED`. A second approve **without** a key on an already approved plan is `409 ALREADY_DECIDED`: it never reserves twice (§59).

## 3. Approval context

Computed from current data at the moment of the call and again, for the stock part, inside the approval transaction:

- `lines`: for each lot of the current revision, `assigned_kg`, the current `available_kg` (on hand − reserved) and `can_reserve` (assigned ≤ available).
- `context_warnings` (all non-blocking, thresholds are UNKNOWN so none blocks): `NEWER_PRICE_DATASET` (a newer non-rejected price dataset than the one the plan used), `STORAGE_CHANGED` (a lot's storage category now differs from the plan's, params lot and from and to), `STOCK_CHANGED_SINCE_PLAN` (a lot's available stock differs from the plan's snapshot).
- `required_acknowledgements`: the distinct codes of the plan's `warnings` and the `context_warnings`.
- `blockers`: `PLAN_NOT_PROPOSED`, `ALREADY_DECIDED`, `NOTHING_TO_RESERVE`, `INSUFFICIENT_AVAILABLE`; `can_approve` is true only without blockers.

## 4. Derived state

`status`: `NOT_PROPOSED` (the plan itself says so), `PROPOSED` (no decision), `APPROVED`, `REJECTED`. `reservation.state`: `NONE` (never approved), `ACTIVE` (remaining > 0), `RELEASED` (approved, remaining 0 and something released). Per line: `reserved_kg`, `released_kg`, `remaining_kg = reserved − released`. Slice 5 will subtract consumption and may add the derived value `CONSUMED`; that extends the enum and will be a new slice version.

## 5. Binding rules

1. **Tenant scope.** The FPO is the principal's. Another FPO's plan answers 404 (audited as a cross-tenant reference). `plan_revisions`, `plan_decisions`, `plan_reservations` and `plan_releases` have row-level security and composite keys to the plan and lots of the same FPO.
2. **Atomicity.** Decision, reservation rows and ledger events commit together or not at all. The decision row's uniqueness per plan serialises concurrent approvals; the ledger trigger serialises competing plans on one lot. Two approvals of two plans that together exceed a lot's stock: exactly one succeeds.
3. **Immutability.** Revisions, decisions, reservations and releases are append-only (trigger, also for the owner).
4. **Audit.** Every decision, edit and release row carries its actor, instant and request ID. Denied attempts are audited by the access layer (`ACCESS_DENIED`). No new audit-event kind is added to `access/v1`.
5. **Explicit failure.** Every refusal has a code; nothing is partly applied.
6. **Exactness.** Quantities are integer grams; the reservation per lot is the exact sum of the approved revision's assignments.

5a. **Order of refusals at approval** (clarification made while building): duplicate decision or replay, plan not proposed, stale revision, nothing to reserve, insufficient stock (`INSUFFICIENT_AVAILABLE`, a hard fact), then unacknowledged warnings (`WARNINGS_NOT_ACKNOWLEDGED`). A decided plan's context carries no context warnings (changes after a decision are not news).

## 6. Not in this slice (deliberate)

"Request a change" from another person (needs a role workflow that is UNKNOWN), approving only some assignments, releasing part of a plan, consumption by sales (slice 5), expiry or same-day cutoff rules (UNKNOWN), blocking freshness thresholds (UNKNOWN), farmer acceptance (UNKNOWN), messages (slice 4), and any use of an `APPROVED` plan by the website's farmer-facing screens.

## 7. Test obligations

Contract examples; backend tests on real PostgreSQL: two FPOs (no read, edit, approval, release or reservation crosses), roles (a viewer or staff cannot approve; staff can edit), the validator rejects every kind of invalid edit and accepts a valid less-fair edit, stale revision, double approval with and without a key, concurrent approvals of one plan from several threads, two plans competing for one lot's stock, release arithmetic, immutability, the ledger shows the reservation and a later plan excludes reserved stock, warnings must be acknowledged.
