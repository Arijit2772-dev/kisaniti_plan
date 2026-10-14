# Conversation amendments and scope limits

Recorded during handoff on 15 October 2026; design conversation/deliverables primarily dated 13–14 October. This is an explicit addendum to the protected August architecture, whose text remains unchanged.

## Selected updates

1. **Hardware selection:** ESP32-WROOM-32 + protected SHT30 I2C probe supersedes DHT22 and the later discarded kits. Preserve the same provider-independent SensorReading/StorageAssessment boundaries and the prohibition on unsupported spoilage claims. Assembly/firmware/field readiness is not proven.
2. **User-provided target context:** harvested paddy/wheat in bags at farm sheds, with electricity and mobile coverage. Preserve per-lot placement, Wi-Fi provisioning and device assignment. This is target/demo context, not a universal real-FPO custody/transport rule.
3. **Automatic monitoring:** recurring manual grain-moisture readings were removed. One-time assembly/provisioning and real sale recording remain distinct needs.
4. **Data design:** broad permitted national market-history acquisition, labelled dummy organization fixtures and four data groups are selected in the Layer 1 baseline. Actual row counts, access, measured bytes and usable forecasting coverage remain unknown until acquisition.
5. **Integration/testing:** every layer, adjacent boundary and complete workflow needs documented acceptance evidence. Physical hardware may arrive later through the same versioned sensor contract. A drawing/simulator/document test does not prove live integration.
6. **Five-layer coverage:** Layer 4/5 review proposals now exist; they extend the architecture with recommendations and tests. Their new permissions, freshness/reconciliation semantics and exact APIs are not automatically frozen merely because the conversation proceeded.

## Unchanged invariants

Multi-FPO isolation, exact inventory/money, provenance, immutable evaluation evidence, computationally isolated optimizer, independent validators, full/partial semantics, per-farmer leximin, versioned policies, UNKNOWN storage, approved-reservation versus actual-sale separation, idempotency/concurrency and explicit failures remain binding.

Forecasting is the primary proposed ML task. No model winner, trained accuracy claim or production solver has been established. GNN is optional research; price training does not occur per sensor/sale/plan event.

## Status and evidence

Evidence for these updates is the preserved [integrated handoff](../handoff/2026-10-14/KisanNiti_Layers_1_to_3_Handoff.md), later Layer 4/5 documents and the user-context record in them. This is a documented conversation summary, not an independently validated FPO interview or complete verbatim chat transcript.

The [current decision status](../blueprint/09_CHAT_DECISION_STATUS.md) states what is selected versus proposed. [Unknown entries](../blueprint/02_UNKNOWN_BLOCKER_REGISTER.md) remain open for real deployment; hardware examples naming DHT22 are historical device examples, not current procurement instructions.

This packaging request did not authorize inventing evidence, silently freezing contracts or claiming implementation. Follow the current blueprint gate and explicitly record any subsequent authorization/status transition before product coding.

## Later clarification on 15 October: WhatsApp and languages

The user explicitly requires farmer communication via WhatsApp and in multiple languages. Current scope is in [the WhatsApp/multilingual specification](../communication/WHATSAPP_MULTILINGUAL.md) and [Layer 5 v0.2](../communication/KisanNiti_Layer_5_Current.md). This supersedes optional/generic-channel wording in the preserved v0.1 snapshot. Simulation remains testing infrastructure. Punjabi/Hindi/English and reviewed templates are recommendations, not a previously confirmed exhaustive language list. No live account or message delivery is established.

## Later decision on 15 October: demo crop focus

After Patiala desk research ([research note](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md)), the user decided that the demo's forecasting and mandi-choice scenarios focus on **wheat held on the farm** and **basmati paddy**. MSP-procured non-basmati paddy and wheat use a **fixed-price path**: the notified MSP from a versioned, labelled MSP_SCHEDULE source, with no forecasting model. This narrows demo scope only; the architecture, contracts and invariants are unchanged. MSP is a policy price, never an observed mandi price or forecast-training row. See CHAT-20 in the [decision status](../blueprint/09_CHAT_DECISION_STATUS.md).
