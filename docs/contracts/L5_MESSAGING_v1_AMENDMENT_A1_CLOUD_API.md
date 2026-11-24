# messaging/v1 amendment A1: WhatsApp Cloud API channel (draft 24 Nov 2026)

**Basis:** CHAT-53 (real WhatsApp required; real sends only to friends' numbers who agreed, on fictional farmers), FULL_PRODUCT_PLAN Phase 4, and L5_MESSAGING_CONTRACT_v1 §4 and §5. That contract already fixes the attempt statuses, the never-backwards rule, the rule that delivery is never evidence of a sale or a consent, and the rule that a real channel refuses an unreviewed template variant. This amendment adds only what those rules leave open.

Parts: **A1.1, A1.2 and A1.3 are built.** The owner decided D1, D3 and the sending number on 24 Nov 2026 (§4).

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

## A1.2 Sending (built 24 Nov 2026; D1 = approved templates)
**Message texts, version 2.** Meta allows no line break inside a template value, so `SALE_INSTRUCTION` and `CANCELLATION` get a version 2. It has the same words as version 1, in all three languages, and differs only in one thing: the sale lines are joined with ` ; ` instead of new lines (owner's choice, 24 Nov).
- Version 2 is `UNREVIEWED` until reviewers confirm the three texts. No review is claimed for it.
- **Version in use** (DESIGN DECISION): messages are prepared with the newest version whose three languages are all `REVIEWED`. Only when no version is fully reviewed is the newest used. So version 1 stays in use, and the preview and the manual WhatsApp handover are unchanged, until version 2 is reviewed. After that, newly prepared messages use version 2.

**Meta's form of a text.** The product's body, with `{{1}}`, `{{2}}`, `{{3}}` in place of its placeholders in the order they appear. `scripts/wa_templates.py` prints the six bodies, sample values and the server setting. The owner creates them in WhatsApp Manager under exactly these names, category Utility:
- `kisaniti_sale_v2`;
- `kisaniti_cancel_v2`.

**Server settings** (from the environment, never git or chat):
- `KN_WA_TOKEN`;
- `KN_WA_PHONE_NUMBER_ID` (digits);
- `KN_WA_API_VERSION` (for example `v23.0`);
- `KN_WA_TEMPLATES`, a JSON map `"<TEMPLATE>.v<version>.<language>": "<meta name>:<meta language code>"`.

Without the first three, or with a malformed value, the channel is unavailable.

**`POST /instructions/{id}/send` with `channel: WHATSAPP`.** Steps in order:
1. Channel not configured: 422 `CHANNEL_NOT_AVAILABLE`.
2. The existing checks: `INSTRUCTION_BLOCKED`, `ALREADY_SENT` (effective status QUEUED or later), and idempotency.
3. The last WhatsApp attempt was `PROVIDER_OUTCOME_UNKNOWN`: 409 `OUTCOME_UNKNOWN`. It is never re-sent automatically, because it may have arrived.
4. The instruction's template is not `REVIEWED`: 409 `TEMPLATE_UNREVIEWED`.
5. No Meta template is set for its template, version and language: 422 `CHANNEL_NOT_AVAILABLE`. A version 1 message is therefore never sent through WhatsApp.
6. **Exact-match check**, else 409 `TEMPLATE_MISMATCH`:
   - the variant's fingerprint must still equal the one stored at preparation;
   - the prepared text must split into values and refill to the identical text;
   - no value may contain a line break, a tab, four spaces, or more than 1024 characters.
7. The send-time eligibility re-check (consent, contact, language, commitment). A blocker gives a `BLOCKED` attempt and no call.
8. One Cloud API call (`POST graph.facebook.com/<version>/<phone number id>/messages`, a template with body parameters, 10 s timeout). The result:
   - 2xx with a message ID: attempt `QUEUED` with `provider_message_id`; later statuses come from the A1.1 webhook;
   - 4xx: `FAILED`, `failure = PROVIDER_REFUSED`, with Meta's numeric error codes only (no provider text). A new attempt is allowed;
   - anything else (5xx, timeout, no connection, no message ID, unreadable answer): `FAILED`, `failure = PROVIDER_OUTCOME_UNKNOWN`. A resend is refused (step 3).

**Stored and enforced.**
- New columns `delivery_attempts.failure` and `error_codes` (migration 0014). The database requires a cause exactly when the status is `FAILED`, and codes only with `PROVIDER_REFUSED`.
- The attempt document gains `failure` and `error_codes` (optional properties in `delivery_attempt.schema.json`).
- The number goes only to Meta. It is never logged.
- Known limit: if the database write fails after Meta has accepted, the message has gone without an attempt row. The log line with the attempt and provider IDs is then the only trace.

## A1.3 Farmers' replies (built 24 Nov 2026; D3 = (a))
Each inbound `messages[]` item in a signed webhook body gets one outcome.

**Matching.** The sender must be a number that one of our WhatsApp messages went to: a `WHATSAPP` attempt with a provider message ID, whose contact version has that number.
- With Meta's reply context (`context.id`) naming one of those messages, the reply belongs to that message's instruction.
- Without a context, it belongs to the newest WhatsApp message sent to that number.
- If messages to that number exist in more than one FPO, a reply without context is `REPLY_AMBIGUOUS` and is not stored (ENGINEERING LEAN: never guess the FPO).
- A context alone never makes an unknown number a farmer.

**Stored.** Only a text of 1 to 500 characters is stored, as a `replies` row:
- `source = WEBHOOK`;
- no user;
- Meta's message ID (unique, so a webhook Meta delivers twice stores the reply once);
- the request ID.

The sender's number is not stored, because the instruction knows its member. A reply is shown only and is never read as agreement (v1 §4).

**Outcomes** (counted in the response, nothing about them logged):
- `REPLY_RECORDED`;
- `REPLY_DUPLICATE`;
- `REPLY_UNMATCHED` (D3: a number we never sent to; only counted, the number goes nowhere);
- `REPLY_AMBIGUOUS`;
- `REPLY_UNSUPPORTED` (not text, empty, or malformed);
- `REPLY_TOO_LONG` (over 500 characters; not stored, known limit).

**Enforced by the database** (migration 0015): a webhook reply has Meta's ID and no user; a staff reply has no Meta ID.

## 4. Decisions (owner, 24 Nov 2026)
- **D1. Message form.**
  - (a) A Meta-approved template per kind and language, with the facts as numbered parameters. The product checks that the filled template equals its own REVIEWED text exactly before sending.
  - (b) Free text inside the 24-hour window, which opens only after the farmer has written to the business number first.
  - Recommended: (a) for the plan instruction; (b) is allowed only for a reply to a farmer's message.
  - **Decided: (a).**
- **D2. Template names and locale codes:** the names are fixed above. The owner creates them; Meta must approve them. The language codes `en`, `pa` and `hi` are what the mapping uses; whether Meta accepts each code for these templates stays **UNKNOWN** until the owner creates them. The server setting is the source, and sending is refused until it is set.
- **D3. Replies from a number that matches no sent message:** (a) refuse and only count them; or (b) store them in a quarantine list visible to the platform operator. Recommended: (a), so a stranger's number is never stored. **Decided: (a).**

- **Sending number (owner, 24 Nov):** Meta's test number. It sends only to up to five numbers verified in the Meta dashboard: friends who agreed, on fictional farmers (CHAT-53).

## Exclusions
Media messages, buttons and interactive replies, more than one business number, and opt-out keywords (a later amendment, together with consent).
