# Decision Register

> **15 October 2026 continuation:** [CHAT-01–CHAT-15](09_CHAT_DECISION_STATUS.md) classify later conversation choices and proposals. Apply the [limited architecture amendments](../architecture/2026-10-15_Conversation_Amendments.md); no production UNKNOWN is closed by the package. Original Phase 0 register text follows as history.

This register records future explicit decisions without replacing the [architecture source of truth](../architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md). It contains no newly invented product or architectural decisions at Phase 0.

Existing architecture decisions remain indexed by their headings and classifications in the source. A future entry must preserve the architecture labels exactly: `CONFIRMED`, `DESIGN DECISION`, `ENGINEERING LEAN`, or `UNKNOWN`.

Do not use this register to resolve an `UNKNOWN` without approved evidence, to revive a revised/rejected idea, or to silently override the architecture. Record conflicts first in [`04_CONFLICT_REGISTER.md`](04_CONFLICT_REGISTER.md).

## Decision entry template

```text
Decision ID:
Area:
Classification:
Status:

Question:

Decision:

Why this decision is needed:

Architecture constraints:

Evidence:

Alternatives considered:

Why selected:

Why alternatives were rejected/deferred:

Failure modes:

Debugging implications:

Testing implications:

Security implications:

Performance implications:

Extensibility implications:

Affected modules/contracts:

Open questions:

Revisit trigger:

Sources:

Date:
```

## Entry acceptance checks

- The question and decision are explicit and small enough to audit.
- Classification and status are not conflated.
- Evidence is recorded in [`03_EVIDENCE_REGISTER.md`](03_EVIDENCE_REGISTER.md) where applicable.
- Affected `UNKNOWN` entries are linked but not deleted; their resolution and status are updated audibly.
- Conflicts and superseded positions are preserved rather than overwritten.

## CHAT-16 / CHAT-17 — WhatsApp and multilingual clarification, 15 October 2026

User explicitly requires WhatsApp farmer messages in multiple languages (DESIGN DECISION). Recommended initial Punjabi/Hindi/English set and deterministic reviewed templates are ENGINEERING LEAN. See [current decision status](09_CHAT_DECISION_STATUS.md) and [communication specification](../communication/WHATSAPP_MULTILINGUAL.md). Platform setup, language review and delivery remain untested; no production unknown is closed merely by naming WhatsApp.

## CHAT-18 / CHAT-19 — storage approximation and Patiala demo assumptions, 15 October 2026

CHAT-18 (ENGINEERING LEAN — proposal): STORAGE-DEMO-0.1 estimates mould/insect-favourable conditions from in-grain SHT30 readings: aw = 24 h median RH/100; LOW < 0.65, MEDIUM 0.65–0.70, HIGH ≥ 0.70; −1 level below 10 °C; LOW→MEDIUM at ≥ 25 °C with aw ≥ 0.60; insect flag advisory only; 6 h settling; 2-point/48 h hysteresis; ambient placement → UNKNOWN. Same rule for paddy and wheat; no moisture % displayed. Experimental, DEMO scope, pending bench/salt/one-time moisture checks.

CHAT-19 (ENGINEERING LEAN — proposal): Patiala DEMO assumptions for CORE-001…009 and PILOT-001…012 as listed in the research note, Part C. All 21 unknowns remain OPEN.

Details: [Patiala research](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md).

## CHAT-20 — demo crop focus and MSP fixed-price path, 15 October 2026

**Classification:** DESIGN DECISION — explicit user decision (resolves conflict PATIALA-MSP-2026-10-15 for the demo).<br>
**Decision:** The demo's price-forecasting and mandi-choice scenarios use (1) wheat held on the farm for later sale and (2) basmati paddy (variety is part of the series identity, e.g. 1121/1509). MSP-procured non-basmati paddy and wheat use a fixed-price path: price = notified MSP for the commodity/grade and procurement season, from a versioned MSP_SCHEDULE source with provenance; no forecasting model runs for them. They still use stock, storage, transport, fairness and approval logic.<br>
**Why:** Desk research shows MSP makes mandi prices nearly identical for those crops in Patiala; prices vary for basmati and for off-season held wheat. See [Patiala research](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md).<br>
**Not decided:** exact MSP_SCHEDULE contract fields, procurement-window dates per season, whether held wheat can also sell to agencies at MSP outside the window, forecast horizons for basmati/held wheat, and real-FPO crop mix.<br>
**Constraints kept:** MSP is a policy price, not an observed mandi price; it must not be added to forecasting training data as observations or counted as forecast accuracy. Missing MSP for a season gives "unavailable", never zero.

## CHAT-21 — SMS fallback for farmers without WhatsApp, 15 October 2026

**Classification:** DESIGN DECISION — explicit user decision (resolves OI-23).<br>
**Decision:** WhatsApp stays the primary farmer channel (CHAT-16). A farmer whose record says they have no WhatsApp gets the same approved instruction as a normal SMS on their mobile number, in their recorded language. SMS uses the same approved assignment facts, recipient checks, permission/opt-out rules, idempotency and delivery tracking as WhatsApp, through the provider-independent CommunicationService.<br>
**Constraints found:** Commercial SMS in India must go through TRAI's DLT system: the sending business (Indian entity with PAN etc.) registers an entity ID, a 6-character sender header and every message template; non-matching messages are blocked ([Plivo DLT guide](https://plivo.com/docs/messaging/concepts/dlt-registration-process), [Telerivet guide](https://www.telerivet.com/blog/india-sms-compliance-trai-dlt-registration-and-tcccpr-guide); vendor sources, re-check at setup). Punjabi/Hindi SMS use Unicode, so fewer characters fit per SMS part and cost rises; templates must be short.<br>
**Open:** SMS provider choice, DLT registration owner (the FPO), template wording/approval, cost, delivery-report reliability. A staff phone call remains the manual last resort and is recorded as such.

## CHAT-22 — price ranges required for longer forecasts, 15 October 2026

**Classification:** DESIGN DECISION — user accepted recommendation (resolves OI-02).<br>
**Decision:** Any price forecast used for horizons longer than 7 days must include a prediction range (at least 80% and 95% intervals, or quantiles), not only a single number. The range is shown to the manager in plain words, and when the range is too wide to support a hold/sell choice the advice says "no reliable signal". Interval quality is evaluated in EXP-04; a model whose ranges are badly calibrated is not promoted for those horizons.

## CHAT-23 — WhatsApp-only assumption; SMS fallback withdrawn, 15 October 2026

**Classification:** DESIGN DECISION — explicit user decision; **supersedes CHAT-21** (kept above as history).<br>
**User words:** "if that is problem in India assume WhatsApp only everyone has".<br>
**Decision:** The demo and current scope assume every member farmer has WhatsApp. No SMS channel is built. A farmer with no usable WhatsApp destination produces a visible "cannot reach" issue for staff follow-up (phone call recorded manually), never a silent skip.<br>
**Why:** Commercial SMS in India needs TRAI DLT registration by a registered business, which a student demo cannot do.<br>
**Risk kept visible:** real WhatsApp coverage among members is an assumption (paper limitation; FPO conversation question). The CommunicationService stays provider-independent so an SMS adapter can be added later without redesign.

## CHAT-24 / CHAT-25 / CHAT-26 — approvals of A1–A3, 15 October 2026

**Classification:** DESIGN DECISION — user approved the recommendations ("okk u doing it plan accordingly").<br>
- **CHAT-24:** Add the **hold-or-sell advisory** to Layer 3 for held wheat and basmati — spec: [HOLD_SELL_ADVISORY](../design/HOLD_SELL_ADVISORY.md). Resolves OI-01 at design level.
- **CHAT-25:** Use the **300-member Patiala persona** for demo, workflow and paper scenarios; keep 5 × 200 (CHAT-02) only as a load test — [FPO profile](../research/PATIALA_FPO_PROFILE_APPROX.md).
- **CHAT-26:** A **privacy contract** (India DPDP Act 2023 / Rules 2025) is required before coding; Aadhaar numbers are never stored — spec: [PRIVACY_CONTRACT](../design/PRIVACY_CONTRACT.md). Resolves OI-06 at design level; legal review still needed before a real pilot.
