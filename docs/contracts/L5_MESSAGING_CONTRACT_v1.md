# L5 slice `messaging/v1` — contacts, consent, templates, instructions, the preview channel and delivery records

26 October 2026 • **Status: FROZEN** (contract version `messaging/v1`) on the owner's approval of slice 4 ("Yes, start slice 4", 26 Oct; scope: S4.1 to S4.4, S4.7, S4.8 of [PRODUCT_FLOW_STATUS.md](../PRODUCT_FLOW_STATUS.md)). Schemas and examples: `kisaniti_implementation/contracts/messaging/v1/`. Builds on `shared/v1`, `org/v1` (members), `plan/v1` and `approval/v1`, all unchanged. Recorded as CHAT-46. Requirements read: [PRIVACY_CONTRACT.md](../design/PRIVACY_CONTRACT.md) (CHAT-26), [WHATSAPP_MULTILINGUAL.md](../communication/WHATSAPP_MULTILINGUAL.md) (CHAT-21/23, acceptance tests WA01 to WA12), [HOSTING_WHATSAPP_HARDWARE_PLAN.md](../HOSTING_WHATSAPP_HARDWARE_PLAN.md) §4. Values marked DEMO are chosen settings, not evidence.

## 0. What this slice is, and is not

An **approved** plan (approval/v1) becomes one **instruction** per farmer: the exact text, in that farmer's recorded language, built from a versioned template and the approved facts, with the contact and consent that allowed it. The **preview channel** shows that text and who it would go to, and sends nothing. Every attempt is a **delivery record**. A manager sees what is ready, what is blocked and why.

**Not in this slice:** a real WhatsApp send (S4.5: needs your Meta account and approved templates), the delivery-status webhook (S4.6: needs a public HTTPS address; the statuses it will write are fixed here), replies, farmer acceptance (UNKNOWN), the erasure and data-access screens of the privacy contract (§8 there; see §6 below), research consent use, and any use of a preview as evidence of delivery. A preview is never `SENT`, `DELIVERED` or `READ`.

## 1. Rules and DEMO policies

| Name | Rule | Class |
|---|---|---|
| Phone numbers | Stored as E.164 in append-only **contact versions** (a change adds a version; the history stays). A 10-digit Indian mobile number (6 to 9 first) is normalised to `+91…`; any other number must already be `+` and 8 to 15 digits. A syntactically valid number is **not** proof that it belongs to the farmer; nothing here claims verification (`source` is `STAFF_ENTERED`). **Fictional numbers only** until the privacy controls and legal review exist | DESIGN (privacy contract) |
| Masking | The API shows a phone **masked** (`+91•••••••210`: the first three characters and the last three digits) unless the caller holds `contacts.view_phone`. Showing an unmasked number writes one row to the append-only `personal_data_access_log` (actor, member, request ID, time). Phone numbers never appear in logs, errors, audit events, instructions or delivery records | DESIGN (privacy contract §2, §6) |
| Consent | Per member and per kind: `ADVICE` (use of data for selling advice and records), `WHATSAPP` (messages), `RESEARCH` (optional). Append-only events `GRANTED` / `WITHDRAWN` with the notice version, the method (`STAFF_ASSISTED_ORAL`, `PAPER_FORM`, `DIGITAL_FORM`) and the staff member. The current state of a kind is its latest event; no event means `NONE`. Withdrawing is as easy as granting and takes effect at the next check. Consent to messages is **not** farmer acceptance of any plan | DESIGN (privacy contract §3) |
| Eligibility to be messaged | An instruction can be prepared for a message only if the member is active, has a current contact, has `WHATSAPP` consent `GRANTED`, and a template variant exists for the member's recorded language. Otherwise the instruction is `BLOCKED` with the issue codes `MEMBER_INACTIVE`, `NO_CONTACT`, `NO_WHATSAPP_CONSENT`, `CONSENT_WITHDRAWN`, `TEMPLATE_UNAVAILABLE`. Language is the member's `preferred_language`; it is never inferred, and there is no silent fallback to another language | DESIGN (WA03) |
| Re-check at send | Immediately before an attempt the same checks run again, plus: the member's language still equals the frozen one (`LANGUAGE_CHANGED`), and for a sale instruction the plan's reservation is still `ACTIVE` (`COMMITMENT_NOT_ACTIVE`). A failed check is recorded as an attempt with status `BLOCKED` and its codes; nothing is sent (WA04) | DESIGN |
| Templates | Versioned files in the product (`SALE_INSTRUCTION`, `CANCELLATION`), each with a Punjabi, a Hindi and an English variant and a per-variant review status (`UNREVIEWED` or `REVIEWED` with reviewer and date). A released version is never edited: a change is a new version. All variants written so far are `UNREVIEWED` (no native-speaker review has happened). The preview shows that label; a real channel (later) must refuse an unreviewed variant. Provider template names and locale codes are UNKNOWN until the Meta account exists and are not part of this slice | DESIGN, review UNKNOWN |
| Rendering | Deterministic and exact: the facts (farmer name, variety, per line quantity in kg with three decimals, mandi name as in the price source, date) are inserted as the stored strings; dates are written with the month name in the variant's language and Western digits (CHAT-37); no price, no transport detail and no guarantee is ever added; a placeholder left unfilled is an error, never sent | DESIGN (WA05) |
| One instruction per farmer | One instruction per (plan, member, kind) holds all of that farmer's lines. Preparing again returns the existing one; only a `BLOCKED` one is replaced (new version, `supersedes` set) once the member is eligible. Languages never produce parallel instructions | DESIGN (WA07) |
| Cancellation | After a plan's reservation was released, a `CANCELLATION` instruction can be prepared for each farmer whose sale instruction has a delivery record with a status of at least `PREVIEWED`; it links the earlier instruction. It says the earlier plan is cancelled and must not be acted on; it never claims the earlier message was recalled | DESIGN (WA08) |
| Authority | Preparing, previewing and viewing instructions: `messages.send` for preparing and sending, `data.read` for viewing. Recording contacts and consents: `data.write`. Viewing an unmasked phone: `contacts.view_phone`. In the demo fixture the FPO admin holds all of them; staff hold `data.write` and may be given `messages.send` | DEMO |

## 2. Objects and endpoints (all under `/api/v1`)

| Object | Schema |
|---|---|
| `ContactV1`, `ContactRequestV1` | `contact.schema.json`, `contact_request.schema.json` |
| `ConsentEventV1`, `ConsentRequestV1`, `ConsentStateV1` | `consent_event`, `consent_request`, `consent_state` |
| `TemplateInfoV1` | `template_info.schema.json` |
| `InstructionV1`, `PrepareRequestV1` | `instruction.schema.json`, `prepare_request.schema.json` |
| `DeliveryAttemptV1`, `SendRequestV1` | `delivery_attempt.schema.json`, `send_request.schema.json` |

| Method and path | Needs | Does | Success | Errors |
|---|---|---|---|---|
| `GET /contacts` | `data.read` (`contacts.view_phone` for unmasked) | every member of the FPO with the current contact (masked) and the current consent state | 200 list of `ContactSummaryV1` | |
| `POST /members/{id}/contact` | `data.write`, CSRF | adds a contact version | 201 `ContactV1` | 404; 422 `PHONE_INVALID`; 409 `NO_CHANGE` |
| `GET /members/{id}/contact/history` | `contacts.view_phone` | all versions, unmasked (logged) | 200 list | 403, 404 |
| `POST /members/{id}/consents` | `data.write`, CSRF | appends a consent event | 201 `ConsentEventV1` | 404; 409 `NO_CHANGE` (the kind is already in that state) |
| `GET /members/{id}/consents` | `data.read` | current state per kind and the history | 200 `ConsentStateV1` | 404 |
| `GET /message-templates` | `data.read` | the templates, versions and the review status of each language | 200 list of `TemplateInfoV1` | |
| `POST /plans/{id}/instructions` | `messages.send`, CSRF | prepares the instructions of an approved plan (or of its cancellation) | 201 list of `InstructionV1` | 404; 409 `NOT_APPROVED`, `NOT_RELEASED`; 422 `NOTHING_TO_CANCEL` |
| `GET /plans/{id}/instructions` | `data.read` | the latest instruction per farmer and kind, with delivery state | 200 list | 404 |
| `GET /instructions/{id}` | `data.read` | one instruction with its attempts | 200 | 404 |
| `POST /instructions/{id}/send` | `messages.send`, CSRF, optional `Idempotency-Key` | re-checks, then makes one attempt on the channel | 201 `DeliveryAttemptV1` (status `PREVIEWED` or `BLOCKED`) | 404; 409 `INSTRUCTION_BLOCKED`, `ALREADY_SENT`; 422 `CHANNEL_NOT_AVAILABLE` |
| `GET /instruction-queue` | `data.read` | instructions that need attention: `BLOCKED`, or last attempt `BLOCKED` or `FAILED` | 200 list | |

Another FPO's member, plan or instruction answers 404 (audited as a cross-tenant reference). `Idempotency-Key`: same key and body returns the stored attempt; another body is `409 IDEMPOTENCY_KEY_REUSED`.

## 3. Delivery states (fixed here, partly used later)

Attempt status: `PREVIEWED` (preview channel; nothing was sent), `BLOCKED` (a check failed; nothing was sent), and, reserved for the real channel and its webhook (S4.5, S4.6): `QUEUED`, `SENT`, `DELIVERED`, `READ`, `FAILED`. Rule for the webhook later: a status never moves backwards (`QUEUED` < `SENT` < `DELIVERED` < `READ`; `FAILED` only before `DELIVERED`), a repeated or out-of-order event changes nothing, and `DELIVERED` and `READ` are messaging evidence only: they never record a sale, a consent or a stock change. An instruction's `delivery.state` is derived from its attempts: `NOT_SENT`, `PREVIEWED`, or the real statuses once they exist.

## 4. Binding rules

1. **Tenant scope.** Contacts, consents, instructions and attempts carry the FPO; row-level security and composite keys as in earlier slices.
2. **Immutability.** Contact versions, consent events, instructions, attempts and the access log are append-only, also for the owner.
3. **Frozen content.** An instruction stores the rendered text, the language, the template id, version and text hash, the review status, the contact version and the consent event that allowed it. Later changes to the contact, language or consent never rewrite it.
4. **No guessing.** Missing contact, consent, language or template is a visible blocker with a code.
5. **Preview is not delivery.** A preview attempt is labelled `PREVIEW — nothing was sent` in its record and can never satisfy a live-delivery check (WA10).
6. **Privacy.** No Aadhaar, bank, land, caste, religion or income field exists; a request carrying any unknown field is refused. Phones are masked unless permitted, never logged, and every unmasked view is logged.

## 5. Test obligations

Contract examples; backend tests on real PostgreSQL: two FPOs (no contact, consent, instruction or attempt crosses; same 404 as a missing one), the three-language rendering with exact quantities and dates for WA01, WA02, WA05, language missing or changed (WA03, WA04), consent withdrawn between prepare and send (PR-T03, WA04), role masking and the access log (PR-T05), no phone in any log line or response except where permitted (PR-T06), unknown fields refused (PR-T01), cancellation linking (WA08), duplicate prepare and duplicate send (WA07), the preview never reporting delivery (WA10), immutability; core tests of phone normalisation, masking and rendering against hand-written expected strings.

## 6. Open (deliberately not decided or built here)

The erasure and data-access screens (PR-T04), a retention job, blocking planning when `ADVICE` consent is missing (PR-T02's second half; it would change plan/v1 behaviour and needs your decision), consent notice texts and their versions (the demo notice version is a name, not a reviewed text), a native-speaker review of the templates, provider template names and locale codes, the real channel, the webhook, shared household phones and representatives.
