# KisanNiti — Layer 4: manager review, decision and commitment

Revision 0.2 • 14 October 2026 • Proposal for review, not implemented or production-frozen. Cross-layer testing and integration acceptance are user-required completion conditions.

Revision 0.2 adds explicit producer/consumer checks, connected scenarios and completion gates. It does not select the still-open business policies or claim application tests have passed.

Continue from the [Layer 1–3 handoff](KisanNiti_Layers_1_to_3_Handoff.md) and [acceptance plan](KisanNiti_Layers_1_to_3_Acceptance.md). This document adds Layer 4 without modifying their historical review/check records or the repository's coding gate.

## 1. Purpose and boundary

Layer 3 proposes a selling plan. Layer 4 lets an authorized FPO manager understand it, change decision variables, reject it, or approve an exact evaluated revision. Approval reserves quantities so another plan cannot commit the same produce.

```text
Evaluated proposal from Layer 3
    → manager reviews evidence and assignments
    → accept / change and revalidate / reject
    → current authority, stock, dates and freshness checks
    → atomic approval + quantity reservations
    → approved commitment handed to Layer 5
```

Approval is not a completed sale, confirmed vehicle booking or proof of farmer acceptance. Those are distinct facts/workflows. Layer 5 owns communication and actual outcomes. Physical hardware never approves plans or changes inventory directly.

No new trained model is required in this layer. It uses application screens, deterministic validation, authorization, exact inventory accounting and audit records.

## 2. Existing decisions versus new recommendations

| Classification | Item |
|---|---|
| DESIGN DECISION, carried forward | Human manager control; tenant isolation; immutable evaluated revisions; current availability recheck; approval/reservation atomicity; idempotency; planned versus actual separation. |
| DESIGN DECISION, carried forward | Manager may change mandi, date and quantity; derived price, distance, assessment and modeled cost cannot be directly overwritten. All revision origins use the same hard validator. |
| DESIGN DECISION, carried forward | Manager can choose a hard-valid alternative that is less optimal unless a policy makes that objective a hard requirement. Validation does not rerun or prove optimizer optimality. |
| ENGINEERING LEAN, new recommendation | Simple demo authority: a designated approver per fictional FPO; optional staff may review and prepare alternatives. No compulsory multi-level approval chain. |
| ENGINEERING LEAN, new recommendation | Three main screens: proposal list, proposal comparison/detail, and approved commitment/history. Approval happens from the reviewed exact revision. |
| ENGINEERING LEAN, new recommendation | Approve an entire evaluated revision atomically. To approve fewer assignments, first create and validate a changed revision; do not approve arbitrary rows invisibly. |
| ENGINEERING LEAN, new recommendation | Until incomplete-solver eligibility is explicitly decided, feasible-but-not-fully-proven solver results remain diagnostic only. Manager-authored hard-valid revisions are a separate path, not an automatic relabeling workaround. |
| UNKNOWN for production | Exact real staff permissions, economic overrides, farmer acceptance, freshness warning/block rules, operational date cutoffs and off-plan sale reconciliation. |

The demo role recommendation is not evidence of how a real FPO operates. Permission checks use authoritative FPO membership/capabilities, never names or email conventions. Backend enforcement is required even if the UI hides a button.

## 3. What the manager sees

### Proposal list

Show FPO, commodity, planning period, selected/allocated/unallocated quantity, proposal revision, generation time, decision state, warnings and whether it is a demo. A proposal state must not imply a delivery or execution state.

### Proposal detail

Each assignment displays farmer, lot, actual/fixture origin, mandi, proposed date, quantity, predicted price and applicable transport basis. Totals display gross expected receipts, modeled transport cost, expected net selling return and remainder. Do not fabricate farmer-level shared-trip cost splits while allocation semantics remain unresolved.

Show:

- Which lots/mandis/dates were considered and which choices were unavailable, with reasons.
- Per-farmer fulfilment and the priorities used, in plain language.
- Storage category or UNKNOWN, its scope, timestamp and explanation; sensor health separately.
- Forecast input cutoff, generation time, model version and applicability; not a guaranteed buyer price.
- Quote validity/applicability and excluded charges; not a confirmed transport booking.
- Validation findings, source age, changed evidence, solver proof status and unresolved issues relevant to approval.
- The exact revision being reviewed. Current facts and frozen proposal evidence remain visibly distinct.

Explanations derive from recorded evidence/constraints. An optional language model is not needed to manufacture reasons. Do not claim an alternative is impossible or worse without a supporting check/comparison.

### Approved commitment/history

Show the approved revision, approver/time, committed quantities and their remaining/consumed/released quantities. Later communication and execution states are separate. Older proposals, rejected alternatives and the original approval remain inspectable.

## 4. Manager actions

| Action | Required behavior | Stock effect |
|---|---|---|
| Review | Open the exact immutable revision and its evidence | None |
| Compare | Compare two evaluated revisions using their recorded inputs; identify differing evidence bases | None |
| Edit proposal | Change decision variables, refresh affected derived inputs, create a new evaluated revision | None |
| Request re-optimization before approval | Run optimizer again with explicitly selected inputs; retain prior run/revision history | None |
| Reject | Record rejection of the exact proposed revision with structured reason/optional note | None |
| Approve | Revalidate current commit conditions; atomically approve exact revision and reserve all its assignment quantities | Reserve, not sell |
| Release remaining commitment | After approval, explicitly select remaining quantities to cancel/release under authority; retain approval and executed history | Release only unexecuted reservation |
| Start new plan after commitment changes | Use current inventory and current evidence in a new planning cycle | None until new approval |

Rejection is for a proposal. Cancellation/release concerns a previously approved commitment. A UI must not pretend these are the same action.

Recommendation: require a concise structured reason for rejection, meaningful manager modification and release, with an optional note. Reasons support investigation; they never automatically retrain models or change policy.

## 5. Editing safely

Example: the proposal assigns 600 kg to Mandi A on Tuesday. The manager requests Mandi B on Wednesday.

1. Preserve the original revision and evidence.
2. Check that B/Wednesday is within the declared search/operating scope. Expanding scope needs an explicit request and evaluation, not a hidden bypass.
3. Obtain the applicable B/Wednesday forecast, route and quote. Recompute costs, totals, constraints and readiness.
4. Run the shared PlanValidator. Missing required evidence or a broken hard constraint prevents approval.
5. Save a new immutable evaluated revision with a link to its predecessor and an explanation of changes. It may show better or worse economics/fairness without claiming optimality.
6. The manager reviews the revised result before approving it. The edited revision must not accidentally use the old mandi's route or price.

Quantity edits must respect availability, selected participating quantities, scheduling increments and full/partial mode. A full-mode requirement cannot quietly become partial. To reduce a plan's scope or change policy, make that an explicit validated change; exact request/revision handling is frozen with the API contracts.

No mandatory optimizer run for a purely manual edit: recomputing derived inputs and validating hard rules is different from optimizing. Only a real solver invocation creates an OptimizationRun.

New recommendation: once a replacement revision is successfully evaluated and selected as the current proposal, mark its replaced proposal SUPERSEDED. Do not destroy the previous proposal because a new edit failed validation. Exact current-revision concurrency handling belongs in contract freeze.

## 6. Approval transaction

### Prepare outside inventory locks

- Resolve exact FPO, plan and revision; verify actor permission and request identity.
- Confirm the revision is proposed and eligible; rejected/superseded/approved revisions cannot be treated as new proposals.
- Validate its frozen hard constraints and evaluation basis.
- Gather required current evidence and compare against the frozen basis. Apply the selected ApprovalFreshnessPolicy.
- Surface any permitted warnings and bind acknowledgment, if required, to the exact revision and checked context. If new evaluation changes decision-relevant amounts, create a new reviewed revision; never edit the old one in place.

### Commit in one short database transaction

1. Recheck current authority as required, revision state, applicable execution dates and policy/context versions.
2. Lock the relevant inventory/reservation records in a defined order and recompute available quantities.
3. Check every required quantity. If any lot conflicts, fail the whole approval without partial reservations; explain affected lots.
4. Create reservations linked to exact assignments and record approval, actor/time and audit context.
5. Persist the durable Layer 5 handoff intent in the same transaction.
6. Commit all or none.

No network request, model inference, solver run or message sending occurs while inventory locks are held. Source-provided expiry must be checked against commit time where applicable; an earlier check does not make an expired quote valid forever.

The local transaction cannot promise that the outside world stops changing. It guarantees consistency with the accepted frozen evidence and current authoritative application state under the agreed policy. Evidence and approval context remain auditable.

### Duplicate and concurrent requests

Proposed command contract includes an idempotency key and expected revision/state version. Repeating the same request returns the existing outcome. Reusing the key with different contents produces a conflict. Two approvers racing must not double-reserve; a losing conflicting request receives a precise stock/revision conflict.

Exact schema, locking/isolation implementation, idempotency retention and retry semantics are implementation-contract tasks. They must preserve the above invariant rather than infer success from a client timeout.

## 7. Freshness and warnings

Historical validity, current stock validity and freshness are different checks. No single “valid for 24 hours” rule is selected for all data.

| Condition | Carried-forward requirement / proposed treatment |
|---|---|
| Wrong FPO or missing approval authority | Hard block; no manager bypass |
| Insufficient currently available stock | Hard block; re-evaluate/replan without silently reducing quantities |
| Rejected/superseded revision | Hard block; create a new evaluated revision if reconsidering |
| Missing required route, forecast or transport economics | Cannot invent a fallback; affected plan needs usable evidence and evaluation |
| Quote explicitly expired | Cannot use it as valid current transport evidence; obtain applicable evidence and re-evaluate |
| More recent forecast/route/quote exists while older evidence may still be usable | Detect separately; exact warning/block/materiality policy remains UNKNOWN for production |
| Sensor missing/stale or crop assessment unsupported | UNKNOWN storage, visible warning; not an automatic allocation/approval ban solely for being UNKNOWN |
| New sensor message with unchanged assessment | No automatic re-optimization or revision overwrite |
| Storage/economic policy or meaningful assessment changed | Evaluate under the selected freshness policy; do not silently replace historical assessment |
| Future operational date no longer valid | Block under explicit date policy; no invented same-day cutoff |

Recommended demo treatment: when refreshed forecast/transport economics or hard policy changes are actually adopted for the candidate, require a newly evaluated revision and renewed review. Sensor-only changes that do not alter the supported category do not force re-optimization. These are recommendations, not final empirical freshness thresholds.

Before implementing approval, freeze a small explicit demo policy table including missing/unavailable evidence, warning acknowledgment, version changes and time boundaries. Keep its scope DEMO. Real-FPO materiality/cutoffs need operational validation; do not copy demo settings into LIVE mode.

## 8. Stock example

| Event | Sold | Remaining reserved | Free to plan from a 1,000 kg lot |
|---|---:|---:|---:|
| Initial lot | 0 | 0 | 1,000 |
| Generate 600 kg proposal | 0 | 0 | 1,000 |
| Approve 600 kg | 0 | 600 | 400 |
| Layer 5 records linked sale of 250 kg | 250 | 350 | 400 |
| Authorized release of remaining 350 kg | 250 | 0 | 750 |

Physical unsold stock and freely available stock differ while reservations exist. A linked sale consumes its reservation exactly once. A new plan can consider the released/current free stock; it cannot rewrite the historical approval. Off-plan sales with active reservations still require the separate unresolved reconciliation workflow.

## 9. Boundary into Layer 5

Layer 4 produces an approved commitment reference containing FPO, exact approved revision, assignment/reservation IDs, approved quantities, approval identity/time and evidence/policy references.

Recommendation: persist a durable pending handoff in PostgreSQL alongside approval. A worker resolves that committed record and passes it to Layer 5 after commit. A database-backed pending-work record is sufficient initially; no new message-platform infrastructure is required by this design.

Layer 5 uses approved assignments for communication, tracks delivery separately, and records actual outcomes. A notification failure does not undo reservations. A crash between approval and worker dispatch must not lose the instruction intent. Duplicate handoffs must not produce duplicate business effects. Later release/cancellation must update the operational instruction workflow so obsolete quantities are not sent blindly; exact delivery/correction behavior is a Layer 5 contract task.

Approval is not authorization to claim the farmer has accepted or a transporter has booked. Real farmer-acceptance requirements remain UNKNOWN. Demo communication can be a sandbox receipt, visibly separate from consent and execution.

## 10. Proposed contracts to freeze

Build on C02 inventory snapshot, C05 readiness, C10 evaluation basis, C13 evaluated proposal and C15 durable jobs from the Layer 1–3 handoff. Names below describe required semantics, not finalized API paths.

| ID | Contract | Required contents |
|---|---|---|
| L4C01 | ProposalReview | Exact revision, authorized scope, assignments/totals/remainder, evaluation basis, validation/proof/readiness and separately labelled current changes |
| L4C02 | RevisionChangeRequest | Expected predecessor/version, explicit changed decisions/scope, reason and idempotency identity; derived facts are not writable fields |
| L4C03 | ApprovalContext | Current authority/stock/evidence checks, policy versions, expiry/date facts, warnings/blockers, context version and any acknowledgment binding |
| L4C04 | DecisionCommand / DecisionOutcome | Approve/reject action, exact revision, expected state/version, idempotency key, actor from authentication, reason where applicable; precise committed/rejected/conflict result |
| L4C05 | ApprovedCommitment | Revision/assignment/reservation references, exact committed quantities, actor/time, audit/evaluation/context references and durable handoff identity |
| L4C06 | ReleaseCommitmentCommand | Selected remaining assignments/quantities, authority, expected versions, idempotency key and reason; exact result and updated remaining reservations |

Canonical tenant/lot/assignment identities and exact numeric conventions must match upstream. An approval response returning “success” must reference a committed record. A network timeout is an unknown client outcome until resolved by idempotent lookup, not evidence the server rolled back.

## 11. Acceptance tests — all NOT_RUN

These extend T38, T40, T43–T46 in the integrated acceptance plan. Tests dependent on unresolved authority/freshness/Layer 5 policy remain blocked until those contracts are selected.

| ID | Scenario | Required result |
|---|---|---|
| L4T01 | Foreign-FPO review/approve/edit/release request | Backend denies access and mutation, including forged referenced IDs |
| L4T02 | Review or reject proposal | No reservation, sale, delivery or model-training side effect |
| L4T03 | Mandi/date edit | New revision uses newly applicable forecast/route/quote, preserves original and passes shared validator |
| L4T04 | Failed edit | Original proposal remains intact; no partially evaluated replacement promoted |
| L4T05 | Edit derived price or exceed stock/increment/full-mode rules | Reject invalid write/plan; no manual bypass of hard invariants |
| L4T06 | Valid but less optimal manager alternative | May pass hard validation under configured policy; no false optimality claim or fabricated solver run |
| L4T07 | Approve 600 from 1,000 | Approved revision plus exactly 600 reserved; 400 free; no ActualSale |
| L4T08 | Double-click/retry after lost response | One committed approval/reservation/handoff; consistent idempotent outcome |
| L4T09 | Same idempotency key, different command | Conflict, not silent reuse or second mutation |
| L4T10 | Concurrent approvals competing for stock | No oversubscription; one succeeds or both fit; otherwise conflict with no partial commitment |
| L4T11 | Failure midway through multi-lot approval | All approval/reservations/handoff intent roll back together |
| L4T12 | Superseded/rejected/already-approved revision | No new approval under invalid state; duplicate approved request handled idempotently |
| L4T13 | Stock/policy/context changed since screen was opened | Recheck at commit; fail or renew review under defined policy, never trust client totals |
| L4T14 | Quote expires between preparation and commit | Cannot commit using now-expired required evidence |
| L4T15 | Warning acknowledgement for different/new context | Stale acknowledgment cannot authorize changed evidence where renewed acknowledgment is required |
| L4T16 | Hardware absent or sensor stale | UNKNOWN visible; no fabricated LOW and no automatic ban solely due to UNKNOWN |
| L4T17 | New market/sensor data after approval | Historical approved revision unchanged; change flag and explicit new-cycle workflow |
| L4T18 | Partial sale then release, repeated release or concurrent sale/release | Only remaining unexecuted reservation released once; exact inventory invariant holds |
| L4T19 | Crash after commit before handoff dispatch; delivery failure | Pending work recoverable; no lost approval or duplicate stock effect; delivery status separate |
| L4T20 | Release occurs before a queued instruction is sent | Layer 5 applies agreed current-commitment/correction rule; cannot blindly send superseded quantities |
| L4T21 | Feasible-but-not-fully-proven optimizer candidate | Eligibility follows explicit policy; diagnostic candidate never silently becomes approved |
| L4T22 | Decision audit and demo isolation | Reconstruct actor/revision/evidence/reason/reservations; no real messages to fixture contacts |

Use real PostgreSQL transactions for concurrency/rollback acceptance, not only mocked repositories. No provider calls inside stock locks. Link exact tested versions and results in completion reports.

## 12. What must be settled next

Layer 4's structure is defined. Before implementation, freeze demo permission capabilities, exact command/state/idempotency contracts, revision replacement behavior, warning/block policy and solver-result eligibility. Before real deployment, validate authority, farmer acceptance, economic overrides and freshness/operational rules with the FPO.

The current recommendation is deliberately simple: one designated demo approver, review an exact evaluated revision, edit through revalidation, approve atomically, then let Layer 5 communicate and record reality. It does not add a new ML model, manual sensor-measurement step or compulsory multi-level approval workflow.

## 13. Integration acceptance is mandatory

Layer 4 is not complete merely because its screens render or its isolated tests pass. Its real application boundaries must agree on IDs, units, policy versions, errors, ownership and side effects. Contracts below reuse the upstream definitions; they must not become separately invented copies.

| Boundary | Required integration behavior | Existing acceptance coverage |
|---|---|---|
| Layer 3 → manager review | C13 becomes L4C01 without losing immutable revision, evidence, remainder, validation or solver-proof information; diagnostic results cannot masquerade as eligible proposals | L4T06, L4T12, L4T21, L4T22 |
| Manager edit → Layer 2/3 evaluation | L4C02 creates a newly evaluated revision; refresh affected derived inputs and reuse the same hard validator; persist original lineage | L4T03, L4T04, L4T05, L4T06 |
| Current facts → approval context | C02/C05 and current evidence/policy versions produce L4C03; expired/conflicting evidence and stale acknowledgments are handled explicitly | L4T13, L4T14, L4T15, L4T16 |
| Decision → inventory + revision state | L4C04 commits approval/reservations/audit/handoff together or none; authoritative server-side permission and idempotency checks apply | L4T01, L4T07, L4T08, L4T09, L4T10, L4T11, L4T12 |
| Approved commitment → Layer 5 | L4C05 identifies a committed exact revision and reservations; durable work survives crashes and downstream failure; no message is sourced from an unapproved proposal | L4T02, L4T19, L4T20, L4T22 |
| Partial execution/release → next plan | L4C06 and the agreed actual-sale contract preserve exact stock, historical approval and current remaining commitments | L4T17, L4T18, L4T20 |

For each boundary, contract tests must exercise valid input, missing/invalid required fields, unsupported schema versions, wrong-FPO references, exact units, duplicate identities and incompatible state/version changes where applicable. The producer's example must be accepted by the actual consumer. Test fixtures and live adapters share the same schema; source origin is retained.

Test the UI and API together: the displayed revision must be the one submitted; pending/failed approval must not appear as success; a lost response must resolve the committed outcome through idempotency; browser refresh must recover server state. Hiding a button is not authorization. Changes to an upstream contract require running the affected downstream checks before merge.

## 14. Connected scenarios to implement

All six scenarios below are **NOT_RUN**. They define test behavior, not completed implementation. Use a fixed clock, seeded fixtures and explicit demo policies for reproducibility. Enable no real external farmer messages.

| ID | Scenario | Expected outcome |
|---|---|---|
| L4I01 | Feed an independently validated Layer 3 proposal for 600 kg of a 1,000 kg lot into review; approve; reload screen | Exactly one approval and 600 kg reservation, 400 kg free, durable pending handoff, no sale; screen agrees with database |
| L4I02 | Change mandi/date, obtain applicable fixture forecast/route/quote, revalidate, review and approve the new revision | New values and revision used; old evidence unchanged; only approved assignments reserved |
| L4I03 | Submit two competing approvals of 600 kg each against the same 1,000 kg lot; retry the winning request after simulating a lost response | At most one succeeds; 600 kg reserved, 400 kg free; retry creates no second effect; loser gets an explicit conflict |
| L4I04 | Cause failure during multi-lot approval, then separately crash after a successful commit before handoff dispatch | First attempt leaves no partial approval/reservation/handoff. Successfully committed attempt remains discoverable and pending work is recoverable |
| L4I05 | Repeat review/approval with simulated readings, then hardware absent/stale; later inject newer market/sensor evidence after approval | Same contracts; UNKNOWN shown when appropriate; no automatic price-model training, no historical approved-revision rewrite, no stock mutation from telemetry |
| L4I06 | Approve 600 kg, record a linked 250 kg sale, release remaining 350 kg, then request a new plan | Sold 250, remaining reservation 0, available 750; duplicate sale/release produces no extra effect; new planning uses current stock |

L4I05's sensor-origin substitution may first use adapter fixtures. Real ESP32 acceptance is a separate bench test and must be reported separately. L4I06 and queued-instruction cancellation cases require the relevant Layer 5 contract before full acceptance; a stub passing only proves the boundary, not Layer 5 implementation.

A failed route refresh need not erase previously stored valid evidence; usability follows the selected freshness policy. Tests must distinguish an unavailable refresh from absence of any usable required evidence. Do not choose a permissive fallback merely to make a test pass.

## 15. Test execution and completion gates

1. **Contract gate:** freeze exact schemas, numeric/time conventions, permissions, demo policy table, eligibility and state/idempotency behavior. Record unresolved LIVE policies as blocked rather than inventing answers.
2. **Component gate:** exercise authorization, revision transitions, deterministic validators, arithmetic, freshness decisions and idempotency outcomes. Testing must check externally meaningful results, not mirror internal implementation.
3. **Database gate:** run reservation/concurrency/rollback tests against actual PostgreSQL. Verify atomicity through persisted rows and audit/handoff records. Check expiry and state changes between preparation and commit. No provider call may occur inside inventory locks.
4. **Adjacent-layer gate:** run the six boundary groups and L4I01–L4I05 using actual application services. Stubs are allowed for controlled external-provider responses, but label them and separately verify real adapters where required.
5. **Full-workflow gate:** after Layer 5 decisions/implementation, run communication, linked partial sale, release, duplicate events, delivery failures and replan, including L4I06. Unresolved off-plan reconciliation remains blocked rather than implemented from guesswork.
6. **Operational gate:** verify restart/recovery, accurate pending/error display, audit reconstruction, tenant isolation and logs without credentials/private farmer details. Record latency at realistic per-FPO workload; no unmeasured response-time claim.

Each completion report must identify code revision, schemas/policies, database/runtime versions, fixture seed/checksum, test procedure, expected/observed outcomes and evidence. Use PASS / FAIL / NOT_RUN / BLOCKED; skipped or mocked-only evidence cannot be reported as a full live pass. Regression checks must run when relevant code/contracts change.

**Definition of done:** all required checks for the claimed release scope pass, known failures are repaired and retested, and real-provider/hardware/full-workflow dependencies are explicitly accounted for. Layer 4 may be labelled integration-ready against an agreed Layer 5 contract before Layer 5 exists, but cannot be labelled end-to-end complete until that integration runs.

Current status: integration design specified; 22 acceptance cases and six connected scenarios defined; application test execution NOT_RUN. This design review provides no guarantee of bug-free operation.
