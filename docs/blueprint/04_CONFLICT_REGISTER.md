# Conflict Register

> **15 October 2026 reconciliation:** historical DHT22/Feather/SHT40/manual-meter kits versus current ESP32+SHT30 are resolved as a hardware-selection supersession only. Collection-centre fixture assumptions versus farm-shed target context are resolved by separate explicit placement profiles. Model winners, storage thresholds and production workflows remain unresolved, not silently selected. See [amendments](../architecture/2026-10-15_Conversation_Amendments.md). Original Phase 0 register text follows.

New real-world evidence that conflicts with the architecture handoff must be recorded here before the architecture is changed. Do not silently reconcile the conflict, reinterpret the handoff, or continue affected work by guessing. Unaffected work may continue only when its safety is explicitly assessed.

No conflicts have been recorded during Phase 0.

## Conflict entry template

```text
Conflict ID:

Date:

Architecture statement:

New evidence:

Evidence source:

Affected decisions/contracts:

Severity:

Can unaffected work safely continue?:

Resolution status:

Resolution:

Architecture update required?:

Notes:
```

## COMMS-2026-10-15 — required channel omitted from Layer 5 handoff

The older architecture names WhatsApp; the Layer 5 v0.1 handoff described generic optional real messaging and omitted multilingual delivery. User clarification resolves required scope: WhatsApp and recipient-language support are required. [Current Layer 5](../communication/KisanNiti_Layer_5_Current.md) supersedes that part of v0.1, which remains preserved as history. The initial language list is a recommendation; no live implementation is claimed.

## PATIALA-MSP-2026-10-15 — MSP procurement flattens mandi price differences for paddy/wheat

**Architecture statement:** the system proposes which mandi to send each lot to, using forecast mandi prices.<br>
**New evidence (desk research, not FPO interview):** in Punjab, non-basmati paddy and wheat are bought by government agencies at MSP (KMS 2025-26 paddy ₹2,369/₹2,389 per quintal; RMS 2026-27 wheat ₹2,585) through ~109 mandis plus temporary yards in Patiala district; May 2026 Punjab wheat mandi average ₹2,597 was within ~0.5% of MSP. Basmati is privately traded and its price varies by mandi.<br>
**Affected:** forecasting value, mandi choice, EconomicPolicy, demo crop selection.<br>
**Severity:** product-scope; architecture unchanged.<br>
**Can unaffected work continue?** Yes.<br>
**Resolution status:** OPEN — user/product decision on demo focus (recommended: held wheat + basmati where prices move; MSP crops via a labelled MSP_SCHEDULE price source).<br>
**Evidence:** [Patiala research](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md), Part A.

**Resolution update, 15 October 2026 (user decision):** the demo focuses on **wheat held on the farm** and **basmati paddy**, where mandi prices actually move. MSP-procured paddy/wheat follow a **fixed-price path**: the notified MSP is a labelled MSP_SCHEDULE price source, not a model forecast. Recorded as CHAT-20. Architecture unchanged; status for the demo scope: RESOLVED. Real-FPO crop mix remains unvalidated.
