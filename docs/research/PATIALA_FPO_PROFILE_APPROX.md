# Approximate Patiala FPO profile (demo persona)

15 October 2026 • **Approximation for the demo, requested by the user.** Built from scheme rules and desk research; no real FPO was contacted. Every value is labelled **SOURCED** (with link), **ESTIMATE** (reasoned, not verified) or **DEMO CHOICE**. The fictional FPO below is a fixture, not a real organisation, and must be shown with the DEMO banner.

## 1. What a Patiala FPO probably looks like

| Aspect | Likely situation | Label |
|---|---|---|
| Number in district | About 11 FPOs listed for Patiala in a July 2024 Rajya Sabha answer (Punjab: 128 under the central scheme) | SOURCED ([Rajya Sabha](https://rsdebate.nic.in/bitstream/123456789/749580/1/PQ_265_26072024_U503_p151_p159.pdf); snippet partial) |
| Legal form | Farmer Producer Company under the Companies Act 2013, or a cooperative | SOURCED (general; [ICAR ATARI](https://ataripune.icar.gov.in/PDF/FPO.pdf)) |
| Members | At least **300** farmer-members (plains rule of the 10,000 FPO scheme), formed from 20+ village groups | SOURCED ([Indian Cooperative](https://www.indiancooperative.com/co-op-news-snippets/minimum-no-of-members-per-fpo-will-be-300-tomar/amp/); [Drishti](https://www.drishtiias.com/daily-news-analysis/formation-promotion-of-10-000-fpos)) |
| Staff | Board of farmer directors; one **CEO/manager** (scheme pays up to ₹25,000/month) and one **accountant** (up to ₹10,000/month) for 3 years; possibly a field coordinator | SOURCED for support caps ([Good Returns](https://www.goodreturns.in/news/pm-kisan-fpo-scheme-here-s-how-farmers-can-get-benefit-up-to-rs-18-lakhs-1226843.html)); staffing pattern ESTIMATE |
| Money | Equity grant up to ₹2,000 per member (max ₹15 lakh); credit guarantee up to ₹2 crore loan | SOURCED ([NCDC SOP](https://www.ncdc.in/documents/employee-corner/42090308212107_SOP_Equity_Grant.pdf); summaries above) |
| Main activities today | Selling inputs (seed, fertiliser) at lower rates; renting machinery (custom hiring, e.g. straw-management machines); some collective marketing | SOURCED as typical FPO functions ([RajRAS](https://rajras.in/agricultural-finance-marketing/)); Patiala-specific mix ESTIMATE |
| Crops | Wheat (rabi) and paddy (kharif) on almost all members' land; some members grow basmati | ESTIMATE (Punjab cropping pattern; Patiala basmati prices observed in Part A of the [Patiala research](2026-10-15_Patiala_Storage_Risk_and_Unknowns.md)) |
| How members sell now | MSP paddy/wheat: each farmer through their own arhtiya at the nearest mandi; basmati: private traders; some wheat kept at home | SOURCED (Patiala research Parts A, C) |
| Digital use | CEO and staff use smartphones and WhatsApp groups; records in registers/Excel | ESTIMATE |
| Storage | Household wheat kept in bags or bins at home/farm sheds; no large FPO warehouse | ESTIMATE |

## 2. Demo FPO fixture built from this profile

**Fictional name:** "Demo Patiala Kisan Producer Company" (DEMO CHOICE; any real-looking name must stay obviously fictional).

| Item | Value | Label |
|---|---|---|
| Members | 300 farmers in 20 villages | DEMO CHOICE (scheme minimum) |
| Members with WhatsApp | 100% assumed (CHAT-23); a few fixture farmers have an invalid number to test the "cannot reach" path | DEMO CHOICE |
| Preferred language | Punjabi 80%, Hindi 10%, English 10% | DEMO CHOICE |
| Land per farmer | Spread from about 1 to 10 ha | ESTIMATE (verify against Punjab agriculture census before using in the paper) |
| Lots per season | Every member: MSP wheat or paddy lot (fixed-price path). About 30% hold some wheat after harvest; about 30% grow basmati | DEMO CHOICE |
| Held wheat per farmer | 10–60 quintals | ESTIMATE |
| Basmati per farmer | 20–150 quintals, varieties 1121/1509/1718 | ESTIMATE |
| Monitored storage | 1 real ESP32 node in one held-wheat or basmati bag; all other lots UNKNOWN or simulated | DEMO CHOICE |
| Candidate mandis | Patiala district mandis such as Patiala, Rajpura, Samana, Patran, Nabha | SOURCED names; distances from the route provider |
| Staff users | CEO = approver; accountant = sale recorder (from J-forms); field coordinator = registration | DEMO CHOICE (PILOT-003) |

A second fictional FPO (smaller, e.g. 150 members) is kept only to test tenant isolation.

## 3. Note for the fixture decision

CHAT-02 selected **5 FPOs × 200 members** as the normal load fixture. The scheme minimum in the plains is **300**, so a realistic persona needs at least 300 members. **Recommendation:** keep CHAT-02 as a pure load test, and use this 300-member persona for all demo, workflow and paper scenarios. **Confirmed by the user on 15 Oct 2026 (CHAT-25).**

## 4. What only a real FPO can confirm

Use one conversation (see architecture §66 and [OPEN_ISSUES](../OPEN_ISSUES.md) OI-16):

- real member count and crops;
- whether members hold wheat, and how much;
- whether the FPO markets basmati collectively;
- who approves decisions;
- who would record sales;
- how many members really use WhatsApp;
- willingness to try advice.
