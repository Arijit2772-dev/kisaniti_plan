# What the user needs to do

15 October 2026 • Things only the project owner can do (accounts, money, people, physical hardware, final choices). Agents keep working on everything else meanwhile. Tick items here and add the date when done.

## A. Decisions to give (quick, in chat)

| # | Decision | Recommended | Why it matters | Ref |
|---|---|---|---|---|
| A1 | ✅ 15 Oct (CHAT-24) Approve the **hold-or-sell advisory** as a new Layer 3 part for held wheat and basmati | Yes | Without it the demo cannot answer "sell now or wait" | [OI-01](OPEN_ISSUES.md) |
| A2 | ✅ 15 Oct (CHAT-25) Use the **300-member Patiala persona** for demo/paper scenarios; keep 5×200 only as a load test | Yes | Scheme minimum is 300 members | [FPO profile §3](research/PATIALA_FPO_PROFILE_APPROX.md) |
| A3 | ✅ 15 Oct (CHAT-26) Approve adding a **privacy (DPDP) contract** before coding; never store Aadhaar numbers | Yes | Legal requirement for real use | [OI-06](OPEN_ISSUES.md) |
| A4 | **Open the coding gate** once contracts are frozen | When ready | Nothing can be built until then | [OI-10](OPEN_ISSUES.md) |
| A5 | Share **UI design ideas** (style, colours, reference apps) | When ready | Frontend look waits for this | [UI plan §6](design/UI_UX_PLAN.md) |

## B. Accounts and access (only you can create these)

| # | Action | Notes | Ref |
|---|---|---|---|
| B1 | Register on **data.gov.in** and request API access for the AGMARKNET (variety-wise) daily price resource | Needed for real price data; never share the key in chat. Store it in a local `.env` file | [OI-05](OPEN_ISSUES.md) |
| B2 | Get access to **CEDA Agmarknet** history (Ashoka University) | Long history for model training | OI-05 |
| B3 | Create a **Meta WhatsApp Business** developer app with a test number | Enough for development and the demo; a real sender later needs the FPO's business verification | [OI-07](OPEN_ISSUES.md) |
| B4 | ~~SMS DLT registration~~ — not needed: WhatsApp only (CHAT-23) | — | CHAT-23 |
| B5 | Decide **hosting budget** (small cloud VM or laptop + tunnel) | Needed for the ESP32 to reach the server | [OI-12](OPEN_ISSUES.md) |

## C. People

| # | Action | Notes | Ref |
|---|---|---|---|
| C1 | **One 30-minute talk** with a Patiala FPO CEO and one arhtiya | Settles most of the 21 unknowns; questions in architecture §66 and the FPO profile §4 | [OI-16](OPEN_ISSUES.md) |
| C2 | Ask your **college guide** about ethics approval and consent forms for any interviews or tests with people | Needed for the paper | [OI-20](OPEN_ISSUES.md) |
| C3 | Confirm the **Springer venue** with your guide (shortlist: SN Computer Science, Agricultural Research, Operations Research Forum, a Springer-proceedings conference) | Shapes length and style | [Springer plan](paper/SPRINGER_PUBLICATION_PLAN.md) |
| C5 | Decide **when the implementation repo becomes public** (needed for code availability at submission) | Springer asks for data/code availability statements | [Springer plan §2](paper/SPRINGER_PUBLICATION_PLAN.md) |
| C4 | Before any real pilot: **legal review** of privacy and messaging | — | OI-06 |

## D. Hardware (physical work)

| # | Action | Ref |
|---|---|---|
| D1 | Buy the one-node kit (ESP32-WROOM-32, protected SHT30 probe, breadboards, resistors, cables, 5 V adapter, about 1 kg wheat) | [Hardware](HARDWARE_CURRENT.md) |
| D2 | Wire it from the circuit drawing; check the probe's wire labels | Same |
| D3 | Salt calibration test (24 h over saturated salt water, expect ~75% RH) | [Patiala research B5](research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md) |
| D4 | Take one grain sample to a mandi moisture meter once and note the reading | Same |

## E. Already done

- Planning folder complete and reviewed; Git set up; private GitHub repos `kisaniti_plan` and `kisaniti_implementation` pushed (15 Oct).
- Demo crop focus (CHAT-20), required price ranges (CHAT-22), WhatsApp only (CHAT-23, replaces CHAT-21 SMS), hold-or-sell advisory (CHAT-24), 300-member persona (CHAT-25), privacy contract (CHAT-26) decided.
- Hold-or-sell spec, privacy contract and Springer publication plan written.
