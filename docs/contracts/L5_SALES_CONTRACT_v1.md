# L5 slice `sales/v1` — replies, actual sales that consume the reservation, reconciliation, planned against actual

26 October 2026 • **Status: FROZEN** (contract version `sales/v1`) on the owner's approval of slice 5 ("Yes, start slice 5", 26 Oct; scope S5.1 to S5.5 and S5.7 of [PRODUCT_FLOW_STATUS.md](../PRODUCT_FLOW_STATUS.md); S5.6, the farmer comprehension study, needs ethics approval and is not built). Schemas and examples: `kisaniti_implementation/contracts/sales/v1/`. Builds on `shared/v1`, `org/v1` (stock ledger), `plan/v1`, `approval/v1` and `messaging/v1`, all unchanged. Recorded as CHAT-47. Architecture sources: handoff §14 (off-plan sale against an active reservation), §53 to §55 (planned vs actual, ActualSale, derived execution state). Values marked DEMO are chosen settings, not evidence.

## 0. What this slice is

Reality comes back. Staff record **actual sales** (from the J-form) and the farmers' **replies**; the system keeps the recommendation and reality as two separate records, derives how far each planned line has been executed, reconciles reservations against sales, and reports planned against actual and the plan's price against the price realised. Nothing here changes the plan, the approval or a message after the fact.

## 1. Rules and policies

| Name | Rule | Class |
|---|---|---|
| ActualSale | Belongs to a **lot**. Fields: lot, actual mandi, actual date, quantity (kg, three decimals), actual price (₹ per quintal, two decimals, positive), J-form number, optional plan reference. The amount is computed by the system: quantity × price ÷ 100, rounded half-even to the paisa, per sale. Never edited or deleted; a mistaken entry is **voided** (§below) | DESIGN (§54) |
| Validation | Quantity > 0; the lot is the caller's FPO's; the mandi exists; the date is not after today (Asia/Kolkata) and not before the lot's harvest date when it has one; the J-form number matches `[A-Za-z0-9][A-Za-z0-9/_-]{0,39}` and is **unique per FPO and lot** (a repeated entry is `409 DUPLICATE_J_FORM`); a sale can never take stock below zero | DESIGN (privacy contract §2: avoid duplicates) |
| `SALE_ASSIGNMENT_CONSUMES_V1` | A sale with a plan reference names an approved plan and one of its lines (`plan_id`, `mandi_id`, `date` of the **planned** line; the actual mandi and date may differ). The plan must be `APPROVED` and the line must exist in the approved revision. The sale **consumes** the plan's reservation of that lot: one `RESERVATION_CONSUMED` and one `SOLD` ledger event, written in this order in the same transaction as the sale, so available stock never changes. The quantity must not exceed what remains reserved for that plan and lot (`409 EXCEEDS_RESERVATION`); the excess is recorded as a separate off-plan sale | DESIGN (§14 "easy case") |
| `OFF_PLAN_MANAGER_CHOOSES_V1` | A sale without a plan reference is always recordable if the lot has the stock. If it is within the lot's **free** stock (on hand − reserved) nothing else happens. If it is more, the request must name which plan reservations to release (`release_reservations`: plan and quantity each, no more than that plan still holds for the lot, together covering the shortfall); the releases and the sale are one transaction. Otherwise `409 RECONCILIATION_REQUIRED` with the candidate plans and what each still holds. More than the lot's on-hand stock is `409 EXCEEDS_STOCK` (correct the lot with a stock adjustment first). The system never chooses which reservation to break | **provisional**: §14 lists this workflow as UNKNOWN to validate with a real FPO before `record_sale` is deployed; it is implemented here as a named, replaceable DEMO policy so that reality can be recorded, and is recorded in CHAT-47 as such |
| Void | A person with approval rights may void a sale with a reason (`ENTRY_ERROR`, `BUYER_CANCELLED`, `OTHER`). The sale stays on record, marked voided; the sold quantity returns to the lot's on-hand stock through a positive `ADJUSTED` ledger event; **a consumed reservation is not restored** (a new plan is needed); voided sales are left out of every execution, report and outcome figure | DESIGN |
| Execution | Derived, never toggled (§55): per planned line `sold_kg` = the lines' non-voided linked sales; `NOT_STARTED` (0), `PARTIAL` (0 < sold < planned), `COMPLETED` (sold ≥ planned); `deviation_kg` = sold − planned when positive. A plan's execution is the weakest of its lines. Off-plan sales on the plan's lots are listed separately | DESIGN |
| Replies | Staff with `messages.send` record a farmer's reply to an instruction (text 1 to 500 characters, as typed, source `STAFF_ENTERED`; the webhook source is reserved). Stored append-only, shown to the same staff, **never parsed, never classified**: a reply does not grant or withdraw consent, accept a plan, record a sale or change stock (privacy contract, WA11) | DESIGN |
| Plan price against realised price | For each planned line with sales: the plan's price (₹ per quintal; forecast + the mandi's basis) against the quantity-weighted average actual price, and the difference. It compares what the plan assumed with what was realised; it is **not** an evaluation of the forecast method, which the pre-registered protocol (C.0) defines. Figures from synthetic (fictional) lots are reported **separately** and must never be used as evidence | DESIGN (CHAT-20 note on synthetic data) |
| Authority | Record a sale: `data.write`. Void: `plans.approve`. Read sales, execution, reports, outcomes: `data.read`. Record or read replies: `messages.send`. | DEMO |

## 2. Objects and endpoints (all under `/api/v1`)

| Object | Schema |
|---|---|
| `ActualSaleV1`, `SaleRequestV1`, `VoidRequestV1` | `actual_sale`, `sale_request`, `void_request` |
| `PlanExecutionV1` | `execution.schema.json` |
| `PlanReportV1` | `report.schema.json` |
| `OutcomesV1` | `outcomes.schema.json` |
| `ReplyV1`, `ReplyRequestV1` | `reply`, `reply_request` |

| Method and path | Needs | Does | Success | Errors |
|---|---|---|---|---|
| `POST /sales` | `data.write`, CSRF, optional `Idempotency-Key` | validates, then writes the sale and its ledger events in one transaction | 201 `ActualSaleV1` | 404 (lot, plan, another FPO's); 409 `NOT_APPROVED`, `EXCEEDS_RESERVATION`, `RECONCILIATION_REQUIRED`, `EXCEEDS_STOCK`, `DUPLICATE_J_FORM`, `IDEMPOTENCY_KEY_REUSED`; 422 `SALE_DATE_IN_FUTURE`, `SALE_BEFORE_HARVEST`, `UNKNOWN_MANDI`, `ASSIGNMENT_NOT_FOUND`, `RELEASE_INVALID`, `PRICE_INVALID`, `QUANTITY_INVALID` |
| `GET /sales` | `data.read` | the FPO's sales, newest first; filters `lot_id`, `plan_id` | 200 list | |
| `GET /sales/{id}` | `data.read` | one sale | 200 | 404 |
| `POST /sales/{id}/void` | `plans.approve`, CSRF | voids it | 201 `ActualSaleV1` | 404; 409 `ALREADY_VOIDED`, `EXCEEDS_STOCK` |
| `GET /plans/{id}/execution` | `data.read` | derived execution and the reservation arithmetic | 200 `PlanExecutionV1` | 404; 409 `NOT_APPROVED` |
| `GET /plans/{id}/report` | `data.read` | planned against actual, price gap, fairness | 200 `PlanReportV1` | 404; 409 `NOT_APPROVED` |
| `GET /forecast-outcomes` | `data.read` | plan price against realised price, real and synthetic separated | 200 `OutcomesV1` | |
| `POST /instructions/{id}/replies` | `messages.send`, CSRF | stores a reply | 201 `ReplyV1` | 404; 422 |
| `GET /instructions/{id}/replies`, `GET /replies` | `messages.send` | replies of one instruction, or the newest of the FPO | 200 list | 404 |

Another FPO's lot, plan, sale, instruction or reply answers 404 (audited as a cross-tenant reference).

## 3. Binding rules

1. **Atomicity.** A sale, its reservation consumption or releases and its ledger events commit together or not at all. The ledger trigger of `org/v1` is the last line of defence: sold + reserved can never exceed what the lot holds.
2. **Separation.** Planned and actual are never merged: sales never edit a plan, a revision or a reservation row; consumption and releases are their own append-only records. Reply text and sale facts are human-entered reality, shown as such.
3. **Reservation arithmetic.** For a plan and a lot: `remaining = reserved − released (by a person) − consumed (by sales) − released for a sale`. `approval/v1` `reservation.state` is `ACTIVE` while remaining > 0; once nothing remains it is `RELEASED` whatever the cause, and `release` is non-null only when a person released it (clarification of the frozen v1: `RELEASED` there means "no reserved stock remains").
4. **Immutability.** Sales, consumptions, releases, voids and replies are append-only, also for the owner.
5. **Privacy.** A reply is personal communication: readable only with `messages.send`, never logged, never in an audit event. No phone number appears in any sales or reply record.
6. **Tenant scope** as in earlier slices.

## 4. Not in this slice (deliberate)

The farmer comprehension and usability study (S5.6), automatic reading of replies or of the J-form image, the delivery-status webhook, a real WhatsApp reply source, transport actuals (so the report compares gross, not net), editing a sale, restoring a consumed reservation, a validated off-plan reconciliation workflow, and any statistical evaluation of forecasts.

## 5. Test obligations

Contract examples; backend tests on real PostgreSQL: a linked sale consumes exactly (ledger events, order, balances, execution), partial and over-line sales, `EXCEEDS_RESERVATION`, an off-plan sale inside free stock, `RECONCILIATION_REQUIRED` and the chosen-release path, `EXCEEDS_STOCK`, every validation refusal, J-form duplicates, void arithmetic, concurrent sales and a sale racing a release (never negative stock), idempotent replay, isolation of all objects, immutability, reply isolation and privacy, execution and report numbers against hand calculation, outcomes separating real from synthetic, mutation checks.
