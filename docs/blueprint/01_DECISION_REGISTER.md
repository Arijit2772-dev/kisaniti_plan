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

## CHAT-27 — research paper first; research prototypes in a separate workspace, 21 October 2026

**Classification:** DESIGN DECISION — explicit user decision ("our main goal right now first is research paper … make different folder kisaniti_research_paper").<br>
**Decision:** Work now goes research-first in `../kisaniti_research_paper` (GitHub `kisaniti_research_paper`, private): research prototypes of every layer, real-data experiments, comparisons and the Springer paper text, written together phase by phase (R0–R10 in its RESEARCH_PLAN.md). Research code is allowed there. The **product coding gate for `kisaniti_implementation` is unchanged** (00_BLUEPRINT_STATUS stays PROHIBITED); research code is not copied into the product without review. Earlier paper v2 work (`~/Documents/rp_kisan`) is imported as the baseline.<br>
**Open decisions listed in the research plan:** D1 hardware story for the paper (recommended ESP32 + SHT30 with Wokwi), D2 venue, D3 prototype dashboard (recommended Streamlit), D4 manuscript format.

## CHAT-28 to CHAT-31 — master plan decisions, 21 October 2026

User words: "we are simultaneously doing implementation also so it would keep on doing things side by side … for the hardware u see what is the best … final software and hardware … ready to use directly". See [MASTER_PLAN](../MASTER_PLAN.md).

- **CHAT-28 (DESIGN DECISION — user decision):** Research and product implementation run side by side. The product coding gate opens **layer by layer**: product code for a layer may start once that layer's contracts are frozen and recorded (W01 slice). The shared algorithm package `core/` lives in `kisaniti_implementation` and is imported by research experiments ("verified once, used twice").
- **CHAT-29 (DESIGN DECISION — choice delegated to Claude by the user):** Hardware for paper and product is ESP32-WROOM-32 + protected SHT30 probe inside the grain, built for real, plus a Wokwi digital twin running the same firmware. The paper v2 DHT22 + 2×DS18B20 circuit is retired (history only). Rationale in MASTER_PLAN §5.
- **CHAT-30 (DESIGN DECISION — delegated):** One frontend — the product Next.js app — supplies the paper screenshots; no separate Streamlit prototype.
- **CHAT-31 (ENGINEERING LEAN — delegated, venue-dependent):** Manuscript in LaTeX with the Springer template unless the venue or guide requires Word.

## W01-L3B — Layer 3b contract slice frozen (device telemetry, SensorReadingV1, StorageAssessmentV1), 22 October 2026

**Classification:** ENGINEERING LEAN — contract freeze delegated by the master plan (M5, Session C) under CHAT-28; no new business policy.<br>
**Status:** FROZEN for implementation. Contract: [L3b_SENSOR_STORAGE_CONTRACT_v1](../contracts/L3b_SENSOR_STORAGE_CONTRACT_v1.md); schemas in `kisaniti_implementation/contracts/sensor/`.<br>
**What is fixed:** message fields and event identity `(device_id, boot_id, seq)`; topics `kisaniti/v1/devices/{id}/telemetry|ack|status`; QoS 1 with backend de-duplication; application acknowledgement after durable store; `STORAGE-DEMO-0.1` constants and order of steps exactly as in Part B (6 h settling, 24 h median, 15 min freshness, 75% coverage, aw 0.65/0.70, 10 °C/25 °C adjustments, 2-point/48 h hysteresis, advisory insect flag).<br>
**Choices made to fill gaps (recorded, reversible by a v2):** (1) hysteresis acts on the water-activity band, and the temperature adjustment is applied after it; (2) an `UNKNOWN` assessment leaves the hysteresis state unchanged; (3) samples taken before NTP sync may be back-filled from uptime within the same boot (`UPTIME_BACKFILLED`) and then count as trusted; (4) firmware v1 buffers 288 samples in **RAM only**, so a power cut loses unsent samples — stated as a limit, flash queue is future work.<br>
**Still open (unchanged):** PILOT-007 local calibration of thresholds, real settling time (bench test H4), broker hosting and device provisioning for a real FPO.

## CHAT-32 — basmati is the main market-priced crop; held wheat becomes a limitation, 22 October 2026

**Classification:** DESIGN DECISION — explicit user decision, based on measured data (refines CHAT-20).<br>
**Evidence (AGMARKNET, Punjab, downloaded 22 Oct 2026):** wheat April 2025 1,081 rows / 98 mandis and May 2025 759 / 83 (procurement, at MSP ₹2,425), but June 2025 only 2 rows / 2 mandis and January 2026 only 5 rows / 3 mandis. Paddy(Basmati) October 2025: 429 rows / 39 mandis, varieties 1509 and 1121, modal ₹2,389–3,765 per quintal. Logged in `kisaniti_research_paper/layers/L1_data/DOWNLOAD_CHECKLIST.md`.<br>
**Decision:** Forecasting, mandi choice and the hold-or-sell advisory are developed and evaluated on **basmati paddy** (by variety). MSP paddy and wheat stay on the fixed-price path. **Held wheat** is not forecast from Punjab mandi data (too sparse); it is reported as a limitation and future work (e.g. open-market series from other states or private-trade data).

## W01-S01 to W01-S08 — shared-basics contract slice frozen (`shared/v1`), 22 October 2026

**Classification:** ENGINEERING LEAN — contract freeze delegated by the master plan (M0, Session B) under CHAT-28. It fixes representations only; no business policy (increments, calendars, economic guardrail, roles) is decided.<br>
**Status:** FROZEN. Spec: [CONTRACT_SPEC_V1_shared](../contracts/CONTRACT_SPEC_V1_shared.md); schemas and valid/invalid examples in `kisaniti_implementation/contracts/shared/v1/`; reference code in `kisaniti_implementation/core/src/kisaniti_core/` (`ids`, `quantity`, `timeutil`, `provenance`, `errors`), tested against the same examples.<br>
- **W01-S01 IDs:** `<kind>_<ULID>` with 14 fixed kinds; byte-string ordering; provider IDs only via mappings; deterministic fixture IDs with time part `0000000000`. *Alternative:* UUIDv4 (not sortable, no visible kind) or provider names (unstable).
- **W01-S02 Tenant scope:** `fpo_id` on every tenant-owned record; the server derives the FPO from the principal; cross-FPO references answer 404 `NOT_FOUND`; mismatched body `fpo_id` answers 422 `TENANT_SCOPE_VIOLATION`; mandis, commodities, market data and MSP are global.
- **W01-S03 Exact numbers:** decimal strings with exactly the type's scale and the unit in the field suffix: kg scale 3 (0–1e9), ₹ scale 2, ₹/kg scale 4, ₹/quintal scale 2, km scale 3. Input is never rounded (`PRECISION_EXCEEDED`); money from quantity × rate is rounded once per line to paise, half to even; totals are sums of rounded lines; ratios are exact fractions. *Alternative:* `{value, unit}` objects (more verbose, same safety) or integer minor units on the wire (unreadable for staff and reviewers).
- **W01-S04 Time:** instants `YYYY-MM-DDTHH:MM:SS.sssZ` only; business dates `YYYY-MM-DD` in the FPO zone; V1 zone `Asia/Kolkata` only (fixed +05:30); inclusive date ranges.
- **W01-S05 Provenance:** `origin` REAL / POLICY / SYNTHETIC, `mode` RESEARCH / DEMO / LIVE; taint rule (any SYNTHETIC input makes the result SYNTHETIC); only REAL counts for accuracy; LIVE refuses SYNTHETIC.
- **W01-S06 Unknown and readiness:** `AVAILABLE` / `UNKNOWN` / `UNAVAILABLE` wrapper (missing is never zero); readiness READY / READY_WITH_WARNINGS / BLOCKED, used by `/health/ready`.
- **W01-S07 Errors:** RFC 9457 problem details with a closed list of 19 codes and fixed HTTP statuses.
- **W01-S08 Versioning:** draft 2020-12 schemas with `.invalid` `$id`s, `contract` field on top-level documents, CI validates every example.<br>
**Revisit trigger:** a second FPO time zone, a unit that needs finer precision than 1 g or 1 paisa, or a real FPO workflow that conflicts with the tenant rule.
