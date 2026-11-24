# messaging/v1 amendment A1: WhatsApp Cloud API channel (draft 24 Nov 2026)

**Basis:** CHAT-53 (real WhatsApp required; real sends only to friends' numbers who agreed, on fictional farmers), FULL_PRODUCT_PLAN Phase 4, and L5_MESSAGING_CONTRACT_v1 §4 and §5. That contract already fixes the attempt statuses, the never-backwards rule, the rule that delivery is never evidence of a sale or a consent, and the rule that a real channel refuses an unreviewed template variant. This amendment adds only what those rules leave open.

Parts: **A1.1 is built now.** A1.2 and A1.3 wait for the decisions in §4.

## A1.1 Status webhook (built; settled by v1 §5)
- `GET /api/v1/webhooks/whatsapp` is Meta's verification. It answers `hub.challenge` as plain text only when `hub.mode=subscribe` and `hub.verify_token` equals the server's `KN_WA_VERIFY_TOKEN`; otherwise it returns 403.
- `POST /api/v1/webhooks/whatsapp` is accepted only with a valid `X-Hub-Signature-256` (HMAC-SHA256 of the raw body with `KN_WA_APP_SECRET`, compared in constant time); otherwise 401 and nothing is stored.
  - With either setting missing, both endpoints answer 503 `CHANNEL_NOT_AVAILABLE`.
  - No session or CSRF: the signature is the authentication.
- Each status in the body (`entry[].changes[].value.statuses[]`, with the provider message ID, `sent|delivered|read|failed`, a Unix timestamp, and error codes) becomes one row in the new append-only table `delivery_events`. The row holds:
  - the attempt it belongs to (found by `provider_message_id` on a `WHATSAPP` attempt);
  - the status;
  - the provider time;
  - the error codes (no text from the provider);
  - the SHA-256 of the raw body;
  - the request ID.
  The recipient number in the event is **not stored**; the attempt already knows its contact version.
- Explicit outcomes per status, counted in the response and logged without phone numbers:
  - `RECORDED`;
  - `DUPLICATE` (the same attempt and status were already recorded; the event is not stored again);
  - `UNKNOWN_MESSAGE` (no WhatsApp attempt has that ID);
  - `UNRECOGNISED_STATUS` (any other value);
  - `IGNORED_BACKWARDS` (stored, but it cannot lower the status).
- The effective status of an attempt is its own row's status (`QUEUED` at send) raised by its events, with the order `QUEUED < SENT < DELIVERED < READ`. `FAILED` applies only while the effective status is below `DELIVERED`. Events that arrive out of order or repeated never lower it. The instruction's `delivery.state` uses the effective status.
- FPO isolation: an event reaches only the FPO that owns the attempt. Every reader still goes through tenant scope.

## A1.2 Sending (waits for D1, D2)
`POST /instructions/{id}/send` with `channel: WHATSAPP` makes the same checks as now, plus these refusals:
- `TEMPLATE_UNREVIEWED`: the variant is not REVIEWED;
- `CHANNEL_NOT_AVAILABLE`: the settings are missing.
It then calls the Cloud API and stores the attempt as `QUEUED` with the provider message ID. A provider refusal is stored as `FAILED` with the error code. The message form depends on D1.

## A1.3 Farmers' replies (waits for D3)
Inbound `messages[]` events become `replies` rows with `source = WEBHOOK`. A reply is shown only and never read as agreement (v1 §4).

## 4. Decisions needed from the owner
- **D1. Message form.**
  - (a) A Meta-approved template per kind and language, with the facts as numbered parameters. The product checks that the filled template equals its own REVIEWED text exactly before sending.
  - (b) Free text inside the 24-hour window, which opens only after the farmer has written to the business number first.
  - Recommended: (a) for the plan instruction; (b) is allowed only for a reply to a farmer's message.
- **D2. Template names and locale codes:** UNKNOWN until the Meta account exists. They are set on the server, never invented; until they are set, sending is refused.
- **D3. Replies from a number that matches no sent message:** (a) refuse and only count them; or (b) store them in a quarantine list visible to the platform operator. Recommended: (a), so a stranger's number is never stored.

## Exclusions
Media messages, buttons and interactive replies, more than one business number, and opt-out keywords (a later amendment, together with consent).
