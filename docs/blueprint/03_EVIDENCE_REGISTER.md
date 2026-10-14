# Evidence Register

> **15 October 2026 evidence index:** later design/source reviews are preserved in [the handoff bundle](../handoff/2026-10-14/README.md). [The amendments](../architecture/2026-10-15_Conversation_Amendments.md) identify conversation context and limits. These are not new runtime, field or source-access experiments. Original Phase 0 register text follows.

Evidence must be attached to a specific decision or question. Different questions require different evidence; collecting material is not itself validation.

## Evidence hierarchy

1. Real FPO workflow evidence.
2. Authoritative domain/government/standards source.
3. Peer-reviewed research.
4. Primary technical documentation/specification.
5. Reproducible KisanNiti experiment/benchmark.
6. Reputable secondary technical literature.
7. Unsupported assumption — not acceptable for freezing production behavior.

Apply the hierarchy according to the question:

- Agricultural/scientific claims require domain literature, authorities, or peer-reviewed research.
- Framework/library behavior requires official technical documentation or specifications.
- Performance choices require reproducible benchmarks in a relevant KisanNiti context.
- FPO workflow and authority questions require real operational validation with the relevant FPO participants.

No citations or evidence claims have been added during Phase 0.

## Evidence entry template

```text
Evidence ID:

Area:

Claim/question supported:

Source type:

Source:

Authors/organization:

Publication/version/date:

What this evidence actually supports:

What it does NOT establish:

Dataset/population/context:

Applicability to KisanNiti:

Limitations:

Reproduction/verification status:

Related decision IDs:

Related unknown IDs:

Confidence:

Notes:
```

## COMMS-E01 — user requirement and platform reference, 15 October 2026

User clarification establishes required WhatsApp/multilingual product scope. [WhatsApp Business Messaging Policy](https://whatsappbusiness.com/policy/) was retrieved on 15 October 2026 (page updated 23 September 2026) and supports recipient opt-in/opt-out and message-template/window requirements. It does not establish any configured account, approved translation, supported exact provider language code, delivered message or application compatibility test. See [communication requirements](../communication/WHATSAPP_MULTILINGUAL.md).

## PATIALA-E01 — Patiala desk research, 15 October 2026

Web desk research covering: FAO storage-fungi water-activity limits (≥65% RH / aw 0.65), FAO insect temperature ranges, inter-granular RH monitoring practice, Patiala climate normals, MSP/procurement rules, digital J-form sale receipts, FCI paddy specifications and district harvesting-time orders. Supports the proposed experimental STORAGE-DEMO-0.1 policy and the DEMO assumptions for the 21 unknowns. **Does not establish:** real-FPO workflow, local storage calibration, transport rates, bag weights or ASABE isotherm constants. Confidence per item is listed in the [research note](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md). No FPO/arhtiya/farmer was contacted.
