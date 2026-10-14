# KisanNiti — Layer 5: communication, actual sales and feedback

Revision 0.1 • 14 October 2026 • Design proposal for review. Not implemented or production-frozen; all application tests below are NOT_RUN.

Continue from the [Layer 1–3 handoff](KisanNiti_Layers_1_to_3_Handoff.md), [Layer 4 proposal](KisanNiti_Layer_4_Review_Proposal.md) and [cross-layer acceptance plan](KisanNiti_Layers_1_to_3_Acceptance.md). Preserve repository governance and the separation of facts, derived information, proposals, approvals and reality.

## 1. Purpose and complete loop

Layer 5 communicates approved commitments, records actual execution, updates authoritative stock through the inventory service and makes outcomes available for the next planning cycle and later evaluation.

```text
Layer 4: exact revision approved + stock reserved + durable handoff
    → resolve current approved commitments
    → prepare/communicate instructions and track delivery
    → authorized source reports actual sales
    → validate and reconcile inventory transactionally
    → current sold/reserved/available quantities
    → manager requests next plan using current facts

Separate evaluation path:
    planned versus actual + reasons → analytics → reviewed model/policy changes
```

Sending an instruction is not a sale. Delivery is not farmer consent. Approval is not a vehicle booking. A sensor reading cannot establish that grain was sold. A manager rejection remains a Layer 4 event; it does not create a sale or deduct stock here.

There is no new required ML model in Layer 5. Its initial work is communication, validated records, exact inventory transactions, audit and analytical comparisons.

## 2. Existing decisions and new recommendations

| Classification | Decision / proposal |
|---|---|
| DESIGN DECISION, carried forward | Communicate from approved assignments; channel adapters never rerun the optimizer. |
| DESIGN DECISION, carried forward | ActualSale belongs to ProduceLot; optional assignment link; an assignment can have multiple actual sales. |
| DESIGN DECISION, carried forward | Preserve planned versus actual mandi/date/quantity/price and estimated versus actual expense separately. |
| DESIGN DECISION, carried forward | Linked sales consume the corresponding reservation transactionally; cancellation releases only unexecuted commitments. |
| DESIGN DECISION, carried forward | Duplicate deliveries/events/sale requests cannot duplicate business effects; delivery failure does not undo approval. |
| DESIGN DECISION, carried forward | Feedback is evidence for investigation, not an automatic retraining trigger or guaranteed proof an optimizer was wrong. |
| ENGINEERING LEAN, new | Start the college demo with a manager-facing instruction view and a simulated delivery adapter. Add real messaging behind the same contract after channel setup and authorization. |
| ENGINEERING LEAN, new | Initial actual-sale source is authenticated staff entry of the real sale outcome; optional import uses the same validation/transaction boundary. A later verified digital receipt integration can replace typing. |
| ENGINEERING LEAN, new | Store unresolved sale reports separately from finalized stock-affecting sales; offer explicit manager-selected reconciliation for conflicting reservations in DEMO, as detailed below. |
| UNKNOWN for production | Actual sale/expense recording authority, farmer acceptance requirement, real messaging provider/access, off-plan/overage reconciliation authority and correction workflow. |

The recommended staff-entry step concerns actual sales, not recurring manual sensor measurements. The chosen hardware still reads temperature/humidity automatically. Without a trustworthy transaction source, the software must not guess sales or mark approved quantities sold automatically.

## 3. Communication

### Input and message content

Consume Layer 4's committed L4C05 handoff. Resolve authorized FPO, exact approved revision, assignment IDs, remaining commitments and applicable communication authority.

Instruction content should identify the intended farmer/recipient, commodity/lot reference, quantity, source location, destination mandi, date and approved operational notes. Include transport arrangements only if actually established. If showing forecast prices, label them as estimates rather than buyer guarantees. Do not expose other farmers' private data in a recipient's message.

Show message content, recipient, instruction version and source approval in the manager view. No real message is sent to fictional contacts. A simulated delivery receipt is visibly labelled simulated, not an actual farmer delivery.

### Durable sending and recovery

1. Read committed pending handoff work; an uncommitted proposal cannot source instructions.
2. Create a versioned instruction referencing approved assignments and current commitment state. Never overwrite the old instruction body after sending.
3. Recheck relevant cancellation/release and destination information before sending. If the instruction is obsolete, suppress/replace pending work under the agreed correction rules.
4. Send outside stock/database locks through a provider-independent adapter.
5. Record attempt identity, provider identity/status, times and failure details. Repeated callbacks are idempotent and authenticated according to the selected provider contract.
6. Retry only safe transient failures with bounded attempts/backoff. Exact values and provider delivery semantics must be frozen before the live adapter is enabled.

Proposed delivery concepts: pending, submitted to provider, delivered where proven, failed and outcome unknown. Keep attempts separately. “Submitted” is not “delivered.” A late failure from an older attempt must not overwrite delivery evidence for a newer successful one. Preserve provider observations instead of inventing certainty from a timeout.

Exactly-once delivery to a person's phone is not promised. If a send times out after the provider may have accepted it, query/reconcile by provider request/message identity where supported before retrying. If outcome cannot be resolved safely, expose it for operator review instead of pretending failure or blindly sending indefinitely.

Delivery failure does not cancel reservations. Show an actionable issue so authorized staff can follow up or explicitly release/replan. No sale is generated by delivery/read/acceptance status.

### Cancellation and instruction corrections

Before a queued instruction is sent, resolve remaining commitment and the latest authorized operational instruction. Suppress a stale instruction if its quantity/date has been cancelled or replaced. Record why it was suppressed.

If an instruction was already sent, preserve the original and create an explicit correction/cancellation notice linked to it. Do not claim the old instruction was recalled or that the farmer received the correction without evidence. If correction delivery is unknown, surface unresolved execution risk to the manager.

There is an unavoidable race between an external send and a concurrent release. Do not hold stock locks while waiting on a messaging provider. Record the instruction/commitment version sent and detect later changes, then issue correction work. Tests must cover this race rather than claim it cannot occur.

## 4. Actual-sale recording

Initial recommendation: an authorized staff member records actual outcomes through the application. Later approved digital receipt/import adapters can use the same backend contract. OCR, if added later, can suggest values but is not an authoritative sale source without its own validation design.

Minimum sale information for a finalized inventory mutation:

- FPO and existing lot identity; actor/source authority supplied by authentication.
- Stable request/event identity for retries and audit.
- Positive exact actual quantity with a known unit.
- Actual sale date and mandi/destination identity under the selected date/location rules.
- Optional valid approved assignment link for the same lot/FPO.
- Actual realized price/rate or total amount, currency and units when known, with evidence references and receipt/recorded time.

Validate relationship and unit consistency. Initial design must not require an achieved price to equal the forecast or the actual date/mandi to equal the recommendation; real deviations are preserved. Future sale claims and ambiguous dates require review under the source's explicit semantics.

Missing financial values must remain missing, never zero or copied from predictions. Recommendation: after authority and inventory validation, an otherwise verifiable sale can deduct its known quantity even if financial settlement is pending; report financial completeness separately. Freeze this behavior and zero/disputed-price semantics before implementation. Do not apply the positive mandi-training-target rule blindly to actual transactions.

Actual transport expense is a separate field/record with its own source, scope and currency. Do not copy the estimate into it. Shared invoice attribution, settlement and payment/accounting are not silently added to V1.

UI outcomes must distinguish “sale report saved for review” from “sale finalized and stock updated.” A database/validation failure cannot be displayed as a successful sale.

## 5. Inventory transaction and partial sales

All stock mutations go through the shared InventoryService. No communication/UI handler directly decrements a lot.

For a normal linked sale within the remaining assigned reservation:

1. Authenticate, validate identifiers/units, and resolve idempotency.
2. In one short transaction, lock/recheck the lot and corresponding reservation.
3. Check the new actual quantity against the applicable remaining reservation and lot invariant.
4. Record finalized ActualSale and consume exactly that quantity from the corresponding reservation.
5. Persist audit and downstream inventory-change work; commit together.

One assignment may execute as several sales. Actual quantities need not be multiples of the optimizer scheduling increment. Duplicate requests create no additional sale or reservation consumption. Record corrections audibly; do not delete history or overwrite stock totals.

```text
available = authoritative lot quantity
            − finalized actual sold
            − remaining active reservations
            − other finalized decrements

remaining reserved = originally reserved − consumed − released
```

The exact signed adjustment model is inherited from the inventory contract; do not subtract the same adjustment twice or invent an editable available-quantity field.

| Event on a 1,000 kg lot | Sold | Reserved remaining | Free to plan |
|---|---:|---:|---:|
| Initial registration | 0 | 0 | 1,000 |
| Approve a 600 kg commitment | 0 | 600 | 400 |
| Linked sale of 250 kg | 250 | 350 | 400 |
| Another linked sale of 100 kg | 350 | 250 | 400 |
| Release remaining 250 kg commitment | 350 | 0 | 650 |
| New planning cycle | 350 | 0 | Up to 650 |

The physical unsold quantity after the first sale is 750 kg, but only 400 kg is unreserved. Do not send 750 kg to the optimizer as freely available stock while 350 kg remains committed.

Execution and commitment closure are separate. A partially fulfilled assignment whose remainder was cancelled must not be described as fully sold. Report sold, remaining reserved and released quantities; freeze exact status labels from these semantics.

## 6. Off-plan sales, overages and corrections

The architecture requires actual off-plan outcomes to be recordable while preserving stock invariants. It forbids silently selecting another approved reservation to break. This workflow remains a production blocker (CORE-009).

**New recommendation for review — explicit reconciliation, not automatic guessing:**

1. Save the reported facts/evidence as a pending outcome report if immediate finalization conflicts with inventory commitments. This preserves the report without falsely claiming a posted sale.
2. Show the exact conflict: reported quantity, physical remaining quantity, free quantity and affected reservations. Do not automatically release the oldest/largest reservation.
3. An actor with explicit reconciliation authority selects which commitments and how much to release, with a reason. The screen must show the operational instruction changes that will be needed.
4. Validate the combined proposed release and sale against current state; commit their inventory/audit/change records atomically. Create durable instruction-correction work for affected commitments.
5. If reported quantity exceeds physical stock even after releases, require investigation of intake, prior sales, duplicates or adjustments. Never invent extra stock merely to accept a report.

An off-plan sale that fits free stock may be posted without breaking other reservations once its authority/recording contract is selected. A linked sale above its remaining commitment needs an explicit overage/reconciliation decision rather than silently consuming another assignment's reservation.

This recommendation is not a claim that a real FPO has approved the workflow. Freeze DEMO authority and command semantics before building it; validate production behavior with the FPO. Until then, preserve conflicting reports as pending and label the workflow blocked. Pending reports must be visible and investigated; they do not disappear into the ledger as confirmed stock truth.

Corrections to finalized sales need a separately specified authorized, versioned reversal/amendment process that preserves references, exact inventory and affected reservations. Do not implement correction as deleting a row or freely adding quantity back. Sale correction and reservation restoration order remain contract-freeze tasks, not assumptions supplied by this proposal.

## 7. Feedback, model evaluation and replanning

Preserve original plan/revision, approval, actual outcomes and reasons for deviations. Examples: different mandi, different date, lower quantity, unavailable transport, farmer decision or corrected record. Reason codes are examples to review, not a validated real-FPO taxonomy.

Keep three evaluations distinct:

- **Mandi forecast accuracy:** compare a saved forecast to the matching later official observed modal price for its target series/date, after target validation. A farmer's achieved sale price is not automatically the same target.
- **Execution and economics:** compare approved versus actual mandi/date/quantity, and modeled versus actual expense where genuinely recorded. Report missing price/cost/grade evidence rather than filling it.
- **Policy/recommendation usefulness:** review fairness, acceptance, remainders and deviations with their context. An override alone does not prove the optimizer/model was wrong; an alternative's unobserved outcome is not known counterfactual profit.

Once a sale/release/adjustment is finalized, update current availability and notify dependent views/jobs. Mark relevant historical proposals as affected where policy requires, but do not rewrite their original evidence or approved decisions.

The manager can request a new plan from current inventory, current usable forecasts, transport evidence and supported assessments. Replanning after commitment changes starts a new planning cycle. Do not blindly reuse the original available quantity or treat still-reserved unsold stock as free.

Source collectors continue on their own schedules. Outcomes can enter analytical datasets, but training/promotion remains a deliberate evaluated process. No automatic retraining per sale, message receipt, rejection or sensor reading; no new-price forecast is required merely because stock changed unless the model's declared inputs actually changed.

## 8. Shared contracts to freeze

Reuse L4C05 approved commitments, L4C06 releases, C02 stock snapshots, C05 validation and C15 durable changes. No provider SDK objects in domain contracts.

| ID | Contract | Required semantics |
|---|---|---|
| L5C01 | InstructionV1 | FPO, approved revision/assignment references, recipient, instruction/commitment version, intended content, real/simulated mode, correction lineage and authorization |
| L5C02 | DeliveryAttemptV1 | Instruction/attempt identity, provider request/message identity, submitted/received event times, authenticated receipt evidence, status/unknown/failure, bounded retry metadata |
| L5C03 | SaleReportV1 | Lot/FPO, optional assignment, source/actor/evidence, exact actual quantity/unit, date/destination, optional financial facts, idempotency identity and report/finalization outcome |
| L5C04 | FinalizedSaleV1 | Canonical sale identity, validated actual facts, consumed reservation references/quantities, inventory version, audit and source report; optional amendment lineage |
| L5C05 | ReconciliationCommandV1 | Explicit selected reservation releases/overage resolution, linked sale report, authority/reason, expected versions and idempotency; atomic outcome; blocked until policy selected |
| L5C06 | ExecutionSummaryV1 | Assignment/lot, approved, sold, remaining reserved, released and available quantities; communication separate; completeness and deviations explicit |
| L5C07 | DecisionOutcomeFeedbackV1 | Planned/actual references, structured reason/notes, actor/time, financial/evidence completeness; no automatic training instruction |
| L5C08 | InventoryOutcomeChangeV1 | Committed sale/release/adjustment identity, affected stock versions and tenant, durable downstream change identity; idempotent refresh/replan notification |

For each: freeze types, precision/units, missing values, date conventions, state transitions, source authority, compatible schema versions and error meanings. Same idempotency key with different contents is a conflict. Similar-looking genuine transactions must not be silently deduplicated solely because they share a date and amount.

## 9. Integration map

| Boundary | Requirement |
|---|---|
| Layer 4 → Layer 5 | Only committed approved references create instructions; one durable handoff identity; delivery failure does not undo approval |
| Layer 5 → shared Layer 2 validation | Apply canonical identities, units, authority, date and relationship checks; sales and market observations use their own contracts |
| Layer 5 → inventory | Finalized sale/reservation consumption/reconciliation atomic; exact derived availability; no duplicate mutation |
| Layer 5 → Layer 3 | New plan uses current available stock and current evidence; old optimization/proposal remains inspectable |
| Layer 1/3 → evaluation | Validate later official target observations; compare to stored forecasts with aligned series/date; actual sale tracked separately |
| Hardware → outcome workflow | Updates storage evidence only; never declares sales, farmer consent or successful delivery |

A database-backed durable job/handoff mechanism is sufficient initially. External delivery calls are outside stock locks. Worker restart, duplicate work, stale queued instructions and uncertain provider outcomes are required scenarios, not optional polish.

## 10. Acceptance cases — all NOT_RUN

These extend T43–T48, L4T18–L4T20 and L4I06. Real-provider tests remain separate from simulated delivery. Reconciliation/correction tests depend on selected policies.

| ID | Scenario | Required result |
|---|---|---|
| L5T01 | Unapproved, foreign-FPO or forged approved reference | No unauthorized instruction, sale, release or data exposure |
| L5T02 | Committed approval handoff replay/restart | One logical instruction intent, preserved lineage, recoverable pending work |
| L5T03 | Simulated delivery to fixture farmer | Clearly simulated receipt; no real external contact |
| L5T04 | Provider submission versus delivery versus acceptance | Distinct states; no invented receipt/consent/sale |
| L5T05 | Transient/permanent send failure and ambiguous timeout | Bounded safe retries; uncertain outcome reconciled/visible; approval/reservation unchanged |
| L5T06 | Duplicate, forged or out-of-order callbacks | Authentication/dedup enforced; older attempt does not erase newer delivery evidence |
| L5T07 | Release before dispatch; release during/after send | Suppress stale pending instruction or create traceable correction; no claim to recall an already sent message |
| L5T08 | Sale input units/date/missing money/actual deviations | Correct source-specific validation; no predicted price substituted; report state differs from finalized sale |
| L5T09 | One assignment sold in 250 kg and 100 kg portions | Two actual sales; corresponding reservation consumption; exact sold/remaining/free amounts |
| L5T10 | Actual-sale quantity not a planning-increment multiple | Record valid exact real quantity under numeric policy; no planning-increment rounding |
| L5T11 | Duplicate sale request/lost response/different payload same key | One stock effect; stable outcome; conflicting content rejected |
| L5T12 | Concurrent sale/release/approval on same lot | Serializable business outcome preserving invariant; no negative or double-committed stock |
| L5T13 | Failure during sale/reservation/audit/change transaction | Complete rollback or complete commit; no sale without its inventory effect |
| L5T14 | Off-plan sale within free stock | Apply selected authority/recording rules; no arbitrary reservation consumption |
| L5T15 | Off-plan sale or linked overage conflicts with reservations | Preserve pending report; explicit validated reconciliation only; no silently selected release |
| L5T16 | Report exceeds physically available stock | Visible investigation/conflict; no fabricated extra stock or false finalization |
| L5T17 | Finalized-sale correction | Block until amendment semantics selected; preserve original and exact corrected inventory/commitments |
| L5T18 | Partial fulfillment then release | Show sold versus cancelled/released accurately; not falsely fully sold; no restoration of already sold stock |
| L5T19 | Expense missing or shared invoice | Missing stays unknown; estimate not copied to actual; no invented farmer allocation |
| L5T20 | New plan after finalized sale/release | Current free stock used; approved historical evidence unchanged |
| L5T21 | Forecast evaluation with actual sale versus official modal target | Correct target separation, series/date alignment and missingness; no unsupported accuracy/profit claim |
| L5T22 | Outcome, delivery callback or sensor update arrives | Correct dependent view updates; no implicit retraining/approval/sale from unrelated event |
| L5T23 | Backup/restore and durable job replay | Audit/history recovered, jobs resume without duplicate sales/reservations |
| L5T24 | Farmer privacy and authority | Recipient sees only authorized content; credentials/private data absent from public logs |

## 11. Whole-application acceptance sequence

Use reproducible fixtures, a fixed clock and explicit DEMO policies. Actual application services and PostgreSQL must participate. Controlled source adapters may be simulated, but mark their evidence as such.

1. Create authorized fictional FPO/member/lot/placement; register 1,000 kg stock. Enable a second FPO to test isolation.
2. Collect and clean market/route/quote/sensor fixtures. Keep real/synthetic origins explicit.
3. Produce a supported forecast or labelled forecast fixture, then optimize/independently validate a proposal for a declared participating quantity of 600 kg.
4. Manager reviews/edits through revalidation if required; approves exact revision; verify 600 reserved and 400 free.
5. Worker prepares an instruction from committed assignments; simulate successful and failed delivery independently; verify no inventory change due to either.
6. Authorized source records linked actual sales of 250 kg then 100 kg; verify sold 350, remaining reservation 250, free 400.
7. Repeat sale request/callbacks and inject a worker crash; verify no duplicate inventory effect and recoverable work.
8. Release the remaining 250 kg under the Layer 4 contract; verify free stock 650, original approval preserved and corrective communication work handled.
9. Request a new planning cycle; selected participating quantities cannot exceed current available stock. Preserve both old and new plan histories.
10. Produce separate reports for forecast-versus-observed-modal evaluation, planned-versus-actual execution and recorded expenses. Missing actual financial values remain missing.

Also run concurrent approval/sale/release, expired quote, missing hardware, schema mismatch, unknown delivery, source outage and off-plan conflict cases. A real messaging/hardware demonstration is a separate gate with authorized destinations and physically observed evidence; a simulator cannot establish it.

## 12. Completion gates and remaining decisions

Before implementation, freeze sale finalization/financial-completeness rules, recording permissions, reconciliation/correction command semantics, instruction version/cancellation rules, source/channel schemas, delivery idempotency/retry behavior and execution-summary definitions. Real messaging provider and credentials are not required to finish the simulated college workflow; do not claim live delivery until that adapter passes its own gate.

Before real-FPO deployment, validate farmer acceptance, sale/expense authority, real quote/expense practice, dispute/correction handling and off-plan reconciliation. None is settled merely by choosing a database or writing tests.

Every claimed release must have recorded unit, contract, real-database concurrency/rollback, adjacent-layer and whole-workflow results, plus applicable provider/hardware tests. Record code/schema/policy/model/firmware versions, input fixtures, expected/observed results and evidence. Use PASS / FAIL / NOT_RUN / BLOCKED; repair and retest failures before completion claims.

Current status: Layer 5 design and integration requirements specified; eight semantic contracts and 24 acceptance cases defined. No new ML model, real messaging, application implementation or software/hardware integration test was executed by this document.
