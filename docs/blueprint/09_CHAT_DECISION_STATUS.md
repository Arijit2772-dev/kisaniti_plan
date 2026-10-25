# Current chat decision status

15 October 2026. Read with the original decision/unknown registers and the dated architecture amendments. “Selected” below means selected design, not implemented-and-tested. The user asked to package all requirements for another agent to refine decisions or start implementation under the applicable gates.

| ID | Classification / status | What to carry forward | Still open |
|---|---|---|---|
| CHAT-01 | DESIGN DECISION — selected | Four source groups: organization, mandi, routes/quotes, sensors | Exact source schemas/API/numeric contracts |
| CHAT-02 | DESIGN DECISION — selected | Dummy normal fixture: 5 FPOs, 1,000 members, 3,000 lots; stress: 50/10,000/30,000 | Generator not built; not real adoption or ML labels |
| CHAT-03 | DESIGN DECISION — selected acquisition strategy | All available permitted national historical market observations; initial 2000-01-01 to 2026-10-12, rolling completed India dates thereafter; recent ten years first | Authorized access, actual rows/date coverage/bytes; historical window not necessarily optimal training window |
| CHAT-04 | DESIGN DECISION — selected starting schedule | Market collection 06:00/18:00 Asia/Kolkata; seven-day correction lookback; sensor every five minutes | Provider limits, measured latency, production freshness policies |
| CHAT-05 | DESIGN DECISION — selected | Separate route estimates from recorded quotes; fixture 200 routes/200 quotes | Actual tariff, grouping, vehicle/trip availability, payer/arranger and expense attribution |
| CHAT-06 | DESIGN DECISION — latest hardware | ESP32-WROOM-32 + protected SHT30; no manual moisture meter; one real node initially | Physical assembly, firmware/broker/durable telemetry and field calibration |
| CHAT-07 | CONFIRMED — user target context | Paddy/wheat bags, farm sheds, power and mobile coverage | Real-FPO custody/verification/dispatch evidence and representative monitoring coverage |
| CHAT-08 | DESIGN DECISION — selected responsibility | Layer 2 cleans/validates all data groups; source lineage and quality/freshness/readiness distinct | Exact numeric scales/schema and empirical screening validation |
| CHAT-09 | ENGINEERING LEAN — candidates | Baselines, XGBoost, Chronos-2 comparison; optional A3T-GCN; MILP/SCIP first prototype | Real evaluation, model promotion, production solver and benchmarks |
| CHAT-10 | DESIGN DECISION — carried forward | Pure mathematical optimizer, exact quantities, per-farmer fairness, independent PlanValidator | Real economic guardrail/increment/calendar and objective acceptance |
| CHAT-11 | ENGINEERING LEAN — proposal | Seven-date demo horizon, partial default, demo market-load disabled, versioned fixture policies | Exact demo policy bundle and live policies |
| CHAT-12 | DESIGN DECISION — required capability | Layer 4 review/edit/reject/approve; approval reserves, not sells | New role model, freshness actions, exact contracts and incomplete-solver eligibility remain recommendations |
| CHAT-13 | DESIGN DECISION — required capability | Layer 5 communicates approved assignments; actual sales update exact stock; outcomes kept separate | WhatsApp account/provider setup, consent/recording authority and correction/reconciliation workflow |
| CHAT-14 | ENGINEERING LEAN — proposal | Simulated delivery for development only (live WhatsApp required); staff-recorded sales initially; explicit manager-selected reconciliation for conflicting reports | Actual authority/workflow acceptance; not a finalized production rule |
| CHAT-15 | DESIGN DECISION — user requirement | Test each layer, contracts, adjacent integration, complete workflow and real hardware separately | Application/physical/evaluation tests not executed |
| CHAT-16 | DESIGN DECISION — explicit user clarification, 15 October | WhatsApp is the required primary farmer channel; messages support recorded preferred language | Account/sender setup, template eligibility, language review and actual delivery tests |
| CHAT-17 | ENGINEERING LEAN — recommendation | Initial Punjabi (Gurmukhi), Hindi, English; reviewed versioned templates, not per-send LLM translation | Final launch language scope, provider locale mapping and approved variants |
| CHAT-18 | ENGINEERING LEAN — proposal, 15 October | Experimental in-grain water-activity storage policy STORAGE-DEMO-0.1 for paddy/wheat | Bench settling time, salt calibration, one-time mandi moisture comparison; user acceptance |
| CHAT-19 | ENGINEERING LEAN — proposal, 15 October | Patiala desk-research DEMO assumptions for all 21 unknowns; MSP finding recorded as conflict PATIALA-MSP-2026-10-15 (demo scope resolved by CHAT-20) | Demo crop-focus decision; confirmation by one FPO CEO + arhtiya |
| CHAT-20 | DESIGN DECISION — user decision, 15 October | Demo focus: held wheat + basmati paddy for forecasting/mandi choice; MSP paddy/wheat via fixed-price MSP_SCHEDULE source (no forecast model) | MSP_SCHEDULE contract, season windows, basmati/held-wheat horizons, real crop mix |
| CHAT-21 | **SUPERSEDED by CHAT-23** — user decision, 15 October | SMS on the farmer's mobile, in their language, when they have no WhatsApp; same approval/permission/delivery rules | SMS provider, TRAI DLT entity/header/template registration by the FPO, cost |
| CHAT-22 | DESIGN DECISION — user accepted, 15 October | Forecasts beyond 7 days must carry 80%/95% ranges; "no reliable signal" when too wide | Interval method chosen by EXP-04 |
| CHAT-23 | DESIGN DECISION — user decision, 15 October | WhatsApp only; every member assumed to have WhatsApp; unreachable farmer = visible staff follow-up; no SMS built | Real coverage among members (paper limitation) |
| CHAT-24 | DESIGN DECISION — user approved, 15 October | Hold-or-sell advisory for held wheat and basmati ([spec](../design/HOLD_SELL_ADVISORY.md)) | Cost parameters; EXP-07 results |
| CHAT-25 | DESIGN DECISION — user approved, 15 October | 300-member Patiala persona for demo/paper; 5×200 only for load tests | Real FPO confirmation |
| CHAT-26 | DESIGN DECISION — user approved, 15 October | Privacy contract (DPDP), no Aadhaar stored ([spec](../design/PRIVACY_CONTRACT.md)) | Legal review before pilot |
| CHAT-27 | DESIGN DECISION — user decision, 21 October | Research paper first: research prototypes + experiments + paper in `kisaniti_research_paper`; product gate unchanged | Decisions D1–D4 in the research plan |
| CHAT-28 | DESIGN DECISION — user decision, 21 October | Research + product side by side; product gate opens layer by layer after that layer's contract freeze; shared `core/` package | Per-layer contract freezes |
| CHAT-29 | DESIGN DECISION — delegated, 21 October | Hardware: real ESP32 + SHT30 in grain + Wokwi twin; DHT22/DS18B20 circuit retired | Kit purchase, calibration |
| CHAT-30 | DESIGN DECISION — delegated, 21 October | One frontend (product Next.js) for paper screenshots | — |
| CHAT-31 | ENGINEERING LEAN — delegated, 21 October | LaTeX Springer template unless venue/guide needs Word | Venue choice |
| CHAT-32 | DESIGN DECISION — user decision on measured data, 22 October | Basmati is the main market-priced crop; off-season Punjab wheat too sparse (2–5 rows/month) → held wheat is a limitation/future work | Basmati season downloads 2021–2026 |
| PAPER-EDITORIAL | DESIGN DECISION (editorial only) — user/supervisor, 22 October | Manuscript in passive/impersonal voice; students first and Harkiran Kaur last; separate citation brackets `[1][4]`; elaborate illustrated figures | Author emails; checker score; venue; redo of the revision (tracker H.1–H.2) |
| CHAT-33 | DESIGN DECISION — user decision, 24 October | ONE paper only; what the data cannot support becomes limitations/future work ([THE_ONE_PAPER](../paper/THE_ONE_PAPER.md)) | — |
| CHAT-34 | DESIGN DECISION — user decision, 24 October | Springer conference format, about 11 pages including references, like the ICDPN 2025 reference paper | Exact conference and page limit with the supervisor |
| CHAT-35 | DESIGN DECISION — user decision, 26 October | **Login is required.** Every user account belongs to an FPO; an FPO is told apart from every other only through the authenticated account, enforced in the backend (never by the UI). Sign-in method, roles and session design are proposed in [AUTH_AND_LANGUAGE_PROPOSAL](../AUTH_AND_LANGUAGE_PROPOSAL.md) and still need approval | Method (password vs OTP), roles and capabilities, account recovery |
| CHAT-36 | DESIGN DECISION — user decision, 26 October | **The web interface is available in Punjabi (Gurmukhi) as well as English.** Confirms OI-13. Interface language is a per-user setting and is separate from each farmer's WhatsApp message language (CHAT-23). Hindi stays optional | Punjabi text is AI-drafted and needs native-speaker review; digit style (Western or Gurmukhi) and default language; font choice |
| DATA-ADEQUACY | ENGINEERING LEAN — 24 October | Variety-level daily index; hold-or-sell exploratory; detectable gain 11–17%; CEDA on hold (no email, no registration, by user choice) | — |

See [current Layer 5](../communication/KisanNiti_Layer_5_Current.md) and [WhatsApp/multilingual specification](../communication/WHATSAPP_MULTILINGUAL.md); earlier optional/generic-channel language is superseded. Storage approximation and Patiala answers: [research note](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md).

## Important unresolved work

- Storage-risk approximation is wanted, but crop thresholds/durations/hysteresis and evidence basis have not been selected or validated. Real telemetry can feed UNKNOWN until a documented policy is ready.
- Deterministic cleaning does not prove reported physical quantities or prices are true. Statistical flags alone do not erase genuine price spikes.
- Schemas in the handoffs specify semantics; they are not completed JSON Schemas, migrations or running APIs.
- Demo settings are not real-FPO evidence. Multiple source kinds may coexist in a visibly labelled DEMO; synthetic rows must not inflate real forecasting evaluation.
- New data updates relevant derived information; it does not retrain every model or rewrite an approved plan.
- Exact cash/quantity conventions, dates, allowed-market mappings, transport economics and outcome reconciliation must be explicit before dependent behavior is coded.

Details and rationale: [Layer 1–3](../handoff/2026-10-14/KisanNiti_Layers_1_to_3_Handoff.md), [Layer 4](../handoff/2026-10-14/KisanNiti_Layer_4_Review_Proposal.md), [Layer 5 current](../communication/KisanNiti_Layer_5_Current.md) (the [v0.1 snapshot](../handoff/2026-10-14/KisanNiti_Layer_5_Review_Proposal.md) is history). Open problems: [OPEN_ISSUES](../OPEN_ISSUES.md). Changes must also be reflected in the formal registers, affected contracts and tests.
