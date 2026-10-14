# WhatsApp and multilingual farmer communication — Layer 5 requirement

15 October 2026. Current user clarification: farmers must be communicated with through WhatsApp and in multiple languages. This document supersedes earlier wording that made live messaging an unspecified optional channel. It adds no authority to send real messages during this documentation task.

## 1. Requirement and scope

> **Superseded by CHAT-23 (WhatsApp only, SMS withdrawn) — kept as history.** Earlier update 15 Oct 2026 (CHAT-21): farmers without WhatsApp receive the same instruction as a normal SMS on their mobile, in their recorded language, via the same CommunicationService rules. Commercial SMS needs TRAI DLT registration (entity, sender header, templates). See the [decision register](../blueprint/01_DECISION_REGISTER.md).

**DESIGN DECISION — explicitly required by the user:** WhatsApp is the primary farmer communication channel in Layer 5. Farmer-facing operational messages must support each recipient's recorded preferred language. The manager application remains the review/administration interface.

A simulated delivery adapter is for development, automated tests and offline demonstrations. It is not a substitute for the required live WhatsApp capability, and simulator-only success cannot satisfy live communication acceptance. The earlier August architecture mentioned WhatsApp; the Layer 5 handoff under-specified it and omitted multilingual delivery. That omission is corrected here without pretending the implementation exists.

**ENGINEERING LEAN — recommended initial language set:** Punjabi (Gurmukhi), Hindi (Devanagari), English. This recommendation follows the Punjab project context; the user has not supplied an exhaustive required-language list. Record each farmer's choice explicitly, allow additions, and never infer language from name, religion, caste, location or phone number. Do not imply all Indian languages are implemented.

**ENGINEERING LEAN — first integration approach:** official WhatsApp Business Platform / Meta Cloud API behind the existing CommunicationService adapter. A supported provider can be used if operational needs justify it; the domain contract stays provider-independent. No personal WhatsApp Web automation or unofficial session scraping is selected. Account configuration, supported API version, sender/number registration, credentials, applicable costs and live approval remain setup tasks.

## 2. Data needed from Layer 1

The authenticated farmer-contact workflow records:

- FPO/member and recipient/contact identity; phone in a consistent international representation; responsible contact and evidence that the destination is the intended recipient.
- Preferred communication language and any explicitly agreed fallback language, with recorded-at/by details. FPO manager UI language is separate.
- WhatsApp messaging permission/opt-in evidence, scope, source/time and opt-out status; distinguish this from acceptance of a particular selling plan.
- Contact/preference version and change history sufficient to explain a previously sent instruction.

Validate input, tenant access and contact/member mapping. A syntactically valid number is not proof of ownership. Missing/unverified contact, missing language or opted-out status produces an actionable communication issue; do not guess or claim delivery. Handling shared household phones/authorized representatives requires explicit mapping and privacy choices, not an automatic merge of farmer records.

## 3. Runtime flow

Approved assignment → current commitment and recipient checks → recorded preferred language → eligible reviewed/approved template variant → fill immutable approved facts → WhatsApp adapter → delivery events → manager-visible delivery state.

1. Use the approved assignment, not optimizer drafts. Preserve revision, lot, farmer, source, destination, date, quantity and units.
2. Resolve current contact/preference/permission and the current remaining commitment immediately before preparing a send. A preference change must not retroactively relabel past messages.
3. Select an eligible template for the message purpose and recipient language. Freeze selected language, template/version and rendered content on the instruction record.
4. Insert validated facts deterministically. Localize labels, dates and unit display under the same exact numeric policy; do not translate or change the amount, destination identity or scheduled date. Use unambiguous dates; no invented transport details or guaranteed sale price.
5. Send via the configured authorized adapter outside database/stock locks; retain provider identifiers and outcome evidence.
6. Interpret delivered/read where actually reported as messaging evidence only. Acknowledgment or a reply is not automatically farmer acceptance or a completed sale.
7. Corrections/cancellations use the same recorded language process, link the previous instruction, and reflect current remaining commitments. Handle release/send races as required by the Layer 5 design.

If the preferred language has no eligible template, queue a visible language/template issue. Use another language only if the recipient has an explicitly recorded acceptable fallback. Never silently send English or another language merely to clear the queue. Real required delivery remains incomplete until a supported path succeeds.

## 4. Translation approach

Use short, versioned, reviewed message templates for common purposes: approved selling instructions; authorized change/cancellation notices; and relevant follow-up/receipt information if that purpose is later selected. Do not expand the system into unsolicited marketing or an autonomous conversational sales agent.

Each language variant must preserve identical semantic fields and meaning. Keep canonical mandi/location IDs and reviewed display names; do not invent transliterations. Long names, Unicode, punctuation, digits, quantities, missing optional fields and date boundaries require preview tests. Omit optional absent information cleanly rather than display fake zero values.

No new runtime translation model is required. AI-assisted drafting of translations is optional authoring support, followed by language review before use. Do not dynamically paraphrase critical quantities, prices, dates or instructions with an LLM on every send. Message text examples are not final provider-approved templates until review/approval is evidenced.

## 5. Platform requirements and evidence limits

WhatsApp's published Business Messaging Policy requires the recipient's contact details and opt-in permission, honoring opt-out, approved templates for business-initiated conversations, and approved templates outside the 24-hour customer-service window. Check current requirements again at adapter setup. These constraints inform eligibility checks; they are not reason to remove WhatsApp from product scope. Source: [WhatsApp Business Messaging Policy](https://whatsappbusiness.com/policy/) (retrieved 15 October 2026; page states updated 23 September 2026).

For routine proactive FPO instructions, plan for reviewed message templates and their provider-approved language variants. Verify exact supported template-language codes and template status for the actual sender/account at implementation; internal language preferences are not automatically valid provider API codes. Current language availability, template approval, account setup and API version were not proven through a live account in this review. Do not treat an archived SDK example as the selected supported runtime.

Consent to receive WhatsApp messages is distinct from farmer agreement to execute a plan. Inbound replies may be retained and surfaced to authorized staff under a separate explicit workflow. No automatic consent, inventory mutation, acceptance classification or sale recording from free-form replies is selected here.

Keep recipient/template account routing tenant-aware. Whether each FPO uses its own sender or an explicitly authorized shared service sender remains an implementation/operational decision; never send under an unrelated FPO's identity.

## 6. Contract amendments

Extend the shared contracts before implementation; these are semantic requirements, not finalized API field syntax:

- Farmer/recipient records: preferred language, explicit fallback, contact verification/source, messaging permission/opt-out and versions.
- L5C01 InstructionV1: channel=WHATSAPP for live farmer instructions, selected language, template identity/version, provider locale mapping, rendered-content identity, permission/contact/commitment versions and source approval. SIMULATED remains a distinct transport mode.
- L5C02 DeliveryAttemptV1: sender/account and provider message identity, language/template references, authentic status events, bounded retry outcome and explicit template/language/contact/permission failures.
- Layer 4 review: manager preview shows recipient, language, exact message and any communication blockers without confusing these with stock approval. No implicit mandatory second human approval for every already-approved eligible send is introduced.
- Layer 5 completion: communication status remains distinct from approval/reservation, farmer acceptance and actual execution; failure does not silently release stock.

Different language variants must not independently duplicate the same instruction. Use recipient + instruction/version identity with the recorded variant and attempt history. Reconcile ambiguous send outcomes before retrying; do not claim exactly-once phone delivery.

## 7. Acceptance tests — all NOT_RUN

| ID | Scenario | Required observation |
|---|---|---|
| WA01 | Three recipients with explicitly selected Punjabi, Hindi and English | Each receives/previews the matching reviewed variant; same assignment facts and exact amounts preserved |
| WA02 | Different language settings for manager UI and farmer | Farmer message follows farmer preference, not manager UI locale |
| WA03 | Missing language or unavailable/paused/rejected preferred template | Visible issue; no silent language switch; explicit accepted fallback only |
| WA04 | Changed contact/language/opt-in before sending; opt-out | Recheck current eligibility; no send to obsolete/unpermitted destination; past instruction audit unchanged |
| WA05 | Mandi names, long strings, Unicode, exact quantities/units and dates | Correct display with no altered canonical meaning or numerical drift; language review recorded |
| WA06 | Proactive message and inside/outside customer-service-window cases | Actual configured provider eligibility and template rules enforced; not fabricated as successful delivery |
| WA07 | Duplicate attempts, repeated/out-of-order webhook or ambiguous timeout | Stable instruction/attempt identity; no duplicate business effect; uncertain outcome visible |
| WA08 | Cancellation/release before, during or after send | Suppression/correction is traceable and uses intended language; no false recall claim |
| WA09 | Cross-FPO recipient/sender/template reference | Unauthorized access/sending rejected; no other farmer data in content |
| WA10 | Simulated adapter versus actual WhatsApp adapter | Same normalized contract; simulation visibly labelled; only real message evidence satisfies live gate |
| WA11 | Reply or read/delivered receipt | Does not automatically record sale, farmer consent, stock change or model training |
| WA12 | Authorized live test recipient for each enabled launch language | Record approved assignment, template/language, provider message ID and actual observed delivery; native-language content review separately documented |

Unit and contract checks cover template selection/field preservation. Integration checks cover Layer 4 handoff → locale/template rendering → adapter/webhook → manager display. Live tests require configured accounts/templates and authorized test recipients. No real messages were sent during this update. WhatsApp and multilingual application tests are NOT_RUN.

## 8. Completion and open setup

The WhatsApp channel and multilingual requirement are selected product scope. The initial language set and direct Meta adapter are recommendations. Account ownership/sender configuration, credentials, exact supported locales, template text/language review/provider approval, costs, live test recipients and permission capture still need completion. Keep these as setup/evidence gates, not reasons to present a simulator-only product as finished.

## 9. SMS fallback acceptance tests (CHAT-21) — WITHDRAWN by CHAT-23 (kept as history; do not implement)

| ID | Scenario | Required observation |
|---|---|---|
| SMS01 | Farmer recorded as "no WhatsApp" with a valid mobile and SMS permission | Same approved instruction sent by SMS in the recorded language; same instruction identity, no duplicate WhatsApp attempt |
| SMS02 | Punjabi/Hindi instruction longer than one Unicode SMS part | Message stays within the approved DLT template; quantity/date/mandi unchanged; part count and cost recorded |
| SMS03 | DLT template missing/rejected or sender header not registered | Visible blocked state; no unregistered send; staff-call fallback suggested |
| SMS04 | Farmer has WhatsApp but delivery fails | No automatic switch to SMS unless the farmer's record allows SMS fallback; decision recorded |
| SMS05 | Duplicate/late SMS delivery reports | One logical delivery state; no duplicate business effect |

## 10. Unreachable farmer (CHAT-23)

| ID | Scenario | Required observation |
|---|---|---|
| WA13 | Farmer has no verified WhatsApp destination, or WhatsApp reports the number cannot receive messages | Visible "cannot reach" issue for staff; manual phone call can be recorded as a separate, staff-entered contact event; never shown as delivered |
