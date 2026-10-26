# L5 slice `handover/v1` — the manager sends from his own WhatsApp: yes/no, one link per farmer

26 October 2026 • **Status: FROZEN** (contract version `handover/v1`) on the owner's approval ("Yes. One tap per farmer is fine until the Meta account exists. Freeze the contract amendment first, then build: OK/approve, Send to farmers, yes/no confirmation, Punjabi message links", 26 Oct). Recorded as CHAT-49 (open item O69). Schemas and examples: `kisaniti_implementation/contracts/handover/v1/`. `messaging/v1` is **unchanged** (a consumer exists, so CHAT-28 makes a change a new version); this is a new slice beside it, using its instructions, contacts and consent. Where it widens one rule of `messaging/v1` (§5 below) the widening is stated, not hidden.

## 0. What this slice is, and is not

An approved plan has one prepared instruction per farmer (`messaging/v1`). The manager presses **Send to farmers**; the page shows who would receive it and who is blocked and asks **yes or no**; on **yes** the system makes one `wa.me` link per eligible farmer: the farmer's number and the **exact stored text** in the farmer's language. The manager taps a link, WhatsApp opens on his own phone with the message ready, and he presses Send there. He then tells the system "sent" or "not sent" for each farmer.

**The system cannot send anything and cannot see what happens in WhatsApp.** No delivery, no read status, no proof. A farmer marked sent is `MANAGER_REPORTED_SENT`: a statement by the manager, never `SENT`, `DELIVERED` or `READ` of `messaging/v1`. Every response carries the label *"Sent by hand from the manager's own WhatsApp. The system cannot see whether it was delivered or read."* One tap sends to one farmer only; sending to everyone from one tap needs the Meta Cloud API (S4.5), which stays not built.

**Not in this slice:** the Meta channel and webhook, free text typed by the manager (a "manager note" is **not** built; it would break the exact-text checks and needs its own decision), group or broadcast-list sending, reading farmers' replies from WhatsApp, any claim of delivery.

## 1. Rules and DEMO policies

| Name | Rule | Class |
|---|---|---|
| Who may | Preview and outcome: `messages.send`. Creating a hand-over (which reveals numbers inside links): `messages.send` **and** `contacts.view_phone`. Viewing a hand-over with links: both again; with only `data.read` the items are shown without links | DEMO fixture (the FPO admin holds all) |
| Eligible farmer | The instruction is `PREPARED` and the checks of `messaging/v1` "Re-check at send" pass now (active member, current contact, `WHATSAPP` consent granted, language unchanged, for a sale instruction the plan's reservation still `ACTIVE`), and the farmer has not already been reported as sent for this plan and kind. Anyone else is listed as blocked with the issue codes, or as already reported | DESIGN |
| The yes | `POST` carries `recipient_count`, the number shown in the question. If the eligible number now differs: `409 CONFIRMATION_STALE`, nothing is created. If no one is eligible: `409 NOTHING_TO_SEND`. The question itself (yes/no) lives on the page; "no" calls nothing | DESIGN |
| Unreviewed text | Every template variant is still `UNREVIEWED` (no native-speaker review). If any eligible farmer's text is unreviewed, the request must carry `acknowledge_unreviewed: true`, else `422 UNREVIEWED_NOT_ACKNOWLEDGED`; the acknowledgement is stored. This replaces the refusal that `messaging/v1` reserves for a real channel, because here the manager reads and sends the text himself | DEMO policy |
| Link | `https://wa.me/<E.164 digits without the plus>?text=<the stored instruction text, percent-encoded UTF-8>`. Built per request from the stored text (its hash must equal the instruction's) and the contact version frozen in the item; **never stored, never logged**. Pattern in `common.schema.json` | DESIGN |
| Phone exposure | Each link returned reveals one number: one row `PHONE_LINK_ISSUED` per farmer per response in `personal_data_access_log` (the `what` list grows by this value; the table stays append-only). A link goes only to a caller with `contacts.view_phone` | DESIGN (privacy contract §6) |
| Checks repeated at view | When a hand-over is viewed, items not yet reported are checked again; if the contact version changed (`CONTACT_CHANGED`) or any other blocker appeared, the link is withheld and `link_withheld` says why. An item reported as sent carries no link; an item reported not sent keeps its link | DESIGN |
| Reports | `SENT` or `NOT_SENT` per item, appended as events (who, when, request ID); the current state is the latest event; a correction is a new event; repeating the current state answers `409 NO_CHANGE`; a blocked item cannot be reported (`409 ITEM_NOT_ISSUED`) | DESIGN |
| Second press | A new hand-over for the same plan and kind includes only farmers not yet reported as sent (for example a farmer whose contact was fixed, or one marked not sent). The earlier hand-overs stay | DESIGN |
| Fictional numbers | Unchanged from `messaging/v1`: only fictional numbers until the privacy controls and a legal review exist; the links of a demo are labelled by the fictional FPO name only | DESIGN |

## 2. Objects and endpoints (all under `/api/v1`)

| Object | Schema |
|---|---|
| `HandoverPreviewV1` | `preview.schema.json` |
| `HandoverRequestV1` | `create_request.schema.json` |
| `HandoverV1`, `HandoverItemV1` | `handover.schema.json`, `item.schema.json` |
| `HandoverOutcomeRequestV1` | `outcome_request.schema.json` |

| Method and path | Needs | Does | Success | Errors |
|---|---|---|---|---|
| `GET /plans/{id}/handover-preview?kind=` | `messages.send` | who is ready, blocked or already reported; counts; no numbers; changes nothing | 200 `HandoverPreviewV1` | 404; 409 `NOT_APPROVED` (sale) / `NOT_RELEASED` (cancellation) |
| `POST /plans/{id}/handovers` | `messages.send` + `contacts.view_phone`, CSRF, optional `Idempotency-Key` | the yes: re-checks, records the hand-over and its items, returns the links | 201 `HandoverV1` | 404; 409 `CONFIRMATION_STALE`, `NOTHING_TO_SEND`, `IDEMPOTENCY_KEY_REUSED`; 422 `UNREVIEWED_NOT_ACKNOWLEDGED`; 403 |
| `GET /plans/{id}/handovers` | `data.read` | the hand-overs of the plan, newest first, never with links | 200 list of `HandoverV1` | 404 |
| `GET /handovers/{id}` | `data.read` (links: `messages.send` + `contacts.view_phone`) | one hand-over with links for items not yet reported and still valid | 200 `HandoverV1` | 404 |
| `POST /handovers/{id}/items/{instruction_id}/outcome` | `messages.send`, CSRF | the manager reports `SENT` or `NOT_SENT` | 201 `HandoverItemV1` | 404; 409 `NO_CHANGE`, `ITEM_NOT_ISSUED`; 422 |

Another FPO's plan, hand-over or instruction answers 404 (audited), as everywhere. Idempotency: the same key and body returns the stored hand-over (with freshly built links); another body is `409 IDEMPOTENCY_KEY_REUSED`.

## 3. Binding rules

1. **Tenant scope and immutability** as in earlier slices: tables carry the FPO, row-level security, composite keys; hand-overs, items and reports are append-only, also for the owner.
2. **No guessing, no fake evidence:** the only delivery-like state is `MANAGER_REPORTED_*`; it never changes `messaging/v1` delivery states, never records a sale, a consent or a stock change.
3. **Exact text:** the link carries the stored instruction text byte for byte; the item stores its hash; a mismatch refuses the link.
4. **No phone number** in logs, errors, audit events or any stored link; only in the response to a permitted caller, with an access-log row.

## 4. Test obligations

Contract examples; backend tests on PostgreSQL: preview (ready, blocked, already reported), the stale count, the unreviewed acknowledgement, link format and exact decoded text for Punjabi, Hindi and English with the number's digits, the access-log rows, a viewer role gets no links and cannot create, two FPOs, reports as events with correction and `NO_CHANGE`, a second hand-over only for the unsent, consent withdrawn and contact changed after the yes, idempotency, immutability, no phone number in log output, cancellation after release; the chain (`scripts/chain.py`) and the website smoke exercise the flow.

## 5. The one rule of `messaging/v1` this slice widens

`messaging/v1` §1 "Cancellation": a `CANCELLATION` can be prepared for each farmer whose sale instruction has a delivery record "of at least `PREVIEWED`". With this slice a farmer whose sale instruction was reported `MANAGER_REPORTED_SENT` also counts as reached. No schema changes; the preparing code reads one more table. Recorded here so the two documents are read together.

## 6. Open

A manager note (free text) on top of an instruction; a reviewed text before real farmers; whether a hand-over to a real FPO needs more than the privacy contract (decided with the legal review); replacing this slice by the Meta channel when the account exists (the instruction and the report remain; the hand-over becomes a fallback).
