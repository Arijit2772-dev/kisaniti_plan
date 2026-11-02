# AGMARKNET hand-check continuation — 28 October 2026

> **Status added 2 Nov 2026:** historical record. Later on 28 Oct the observations were transferred to the research repository and the owner signed off (research `results/L2_cleaning/hand_check_signoff.json`, `layers/L2_cleaning/CLEANING_EVIDENCE.md` §6, `results/audit/AUDIT_2026-10-28.md`). Do not redo the continuation steps or treat "pending" below as current. Still open: B.2b second-source check; arrivals meaning UNKNOWN.

Scope: B.2 source inspection only; no cleaning-rule change, dataset validation, or human sign-off. Checked by Codex using the live Chrome report after the user submitted the CAPTCHA. Research sibling is outside this session's writable roots; its hand-check sheet remains unchanged. This note preserves the observation for reconciliation there.

## HC26 — November 2023

Source: https://agmarknet.gov.in/datewisespeccommodityoutput
Report title verified: November 2023 / Paddy(Basmati) / Punjab.
Market: Abohar APMC. Target variety: Basmati 1509. Intended date: 14 November 2023.

| Field | Saved hand-check sheet | Live report |
|---|---|---|
| Date | 2023-11-14 (inferred) | Blank cell immediately following dated 14/11/2023 row, within Abohar |
| Arrivals, metric tonnes | 700 | 350.00 |
| Minimum, INR/quintal | 3000 | 3000.00 |
| Maximum, INR/quintal | 3800 | 3800.00 |
| Modal, INR/quintal | 3600 | 3600.00 |

Read-only DOM inspection: preceding 1121 row has date 14/11/2023 and arrivals 350.00. Target row has six cells, including an empty date cell; neither row has a rowspan attribute. Therefore continuation beneath the preceding date is observed, but a merged date cell is NOT observed. Prices agree; full-row agreement is NO due to arrivals. Meaning of the arrivals difference remains UNKNOWN.

Original research raw file `data/raw/agmarknet/paddy_basmati/2023-11_punjab.csv`, lines 24–25, inspected unchanged:

```csv
14/11/2023,700.0,1121,4000.0,4270.0,4111.0
,,Basmati 1509,3000.0,3800.0,3600.0
```

Possible total-versus-per-variety presentation difference is a hypothesis, not an approved division rule. Do not change historical raw data or mark all blank-date rows validated from this observation. This agent inspection is not the owner's recorded human hand-check sign-off or an independent second-source check.

## Next

Prepare December 2023 / Punjab / Paddy(Basmati) for human CAPTCHA. HC27: Budalada APMC, Basmati 1509, 9 December 2023; sheet arrivals 100 tonnes, min 3800, max 5000, modal 4800 INR/quintal. Earlier HC01–HC25 remain unverified in the saved sheet; no completion inferred from the user's recollection of the prior stopping point.

## HC27 — December 2023 inspected

Codex inspected the live December 2023 / Paddy(Basmati) / Punjab report after human CAPTCHA submission on 28 October 2026. Under Budalada APMC, the 09/12/2023 Basmati 1509 row shows arrivals 100.00 tonnes, minimum 3,800.00, maximum 5,000.00 and modal 4,800.00 INR/quintal. All fields match HC27. This is agent source inspection, not owner sign-off. Research sheet unchanged due to workspace boundary.

Next selected sample: HC28, September 2024, Attari APMC, Basmati 1509, 23 September 2024; sheet arrivals 103 tonnes, min 2250, maximum missing, modal 2646. Months without selected sample rows are skipped.

## HC28 — September 2024 inspected

28 October 2026, Codex live-page inspection after human CAPTCHA: report title September 2024 / Paddy(Basmati) / Punjab; Attari APMC, 23/09/2024, Basmati 1509. Arrivals 103.00 tonnes, min 2,250.00, max 0.00, modal 2,646.00 INR/quintal. Date, arrivals, min and modal match HC28; maximum differs from the sheet's missing value. Full agreement NO. The website's displayed zero is observed, not a valid replacement for missing data: maximum 0 is inconsistent with positive min/modal. Preserve missing-value handling pending explicit review. No raw changes or dataset validation. Next HC29: October 2024, Garh Shankar APMC, 1121, 20 October; arrivals 3560, min/max/modal 2320.

## HC29 — October 2024 inspected

28 October 2026, Codex live-page inspection after human CAPTCHA: report title October 2024 / Paddy(Basmati) / Punjab; Garh Shankar APMC, 20/10/2024, 1121. Arrivals 3,560.00 tonnes; min/max/modal each 2,320.00 INR/quintal. All HC29 fields match. Source agreement does not establish correct commodity classification or economic accuracy of the constant price. Research sheet unchanged; owner sign-off pending. Next HC30: November 2024 / Rayya APMC / Basmati 1509 / inferred 18 November; sheet arrivals 42, min 2600, max 3200, modal 2800.

## HC30 — November 2024 inspected

28 October 2026, Codex inspected the live November 2024 / Paddy(Basmati) / Punjab report after human CAPTCHA. Rayya APMC: dated 18/11/2024 1121 row shows 21.00 tonnes; following Basmati 1509 row has empty date cell, 21.00 tonnes, min 2600, max 3200, modal 2800 INR/quintal. No rowspan attribute on either row. Prices match HC30; arrivals differ from sheet 42 tonnes, so full agreement NO. Continuation is observed, merged date cell is not. The repeated half-total pattern (also HC26) warrants review of export-versus-display arrivals semantics; no division rule adopted. Raw data and research sheet unchanged; owner sign-off pending.

Next: December 2024, HC31–HC35 (Attari, Fazilka, Khanna, Nawanshahar); inspect all five within one report.

## HC31–HC35 — December 2024 inspected

28 October 2026, Codex live report after human CAPTCHA; title December 2024 / Paddy(Basmati) / Punjab verified. Units: tonnes and INR/quintal. Agent inspection only; research sheet unchanged and owner sign-off pending.

| Check | Market / date / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC31 | Attari / 01 Dec / Basmati 1509 | 24.00 | 4100 / 4300 / 4200 | All fields match |
| HC32 | Fazilka / 12 Dec / 1121 | 2000.00 | 3400 / 3925 / 3500 | All fields match |
| HC33 | Khanna / 09 Dec / 1121 | 146.00 | 2550 / 4330 / 3200 | All fields match |
| HC34 | Nawanshahar / continuation after 04 Dec / 1121 | 34.88 | 3475 / 4221 / 3700 | Prices match; arrivals differ from sheet 69.75; full agreement NO |
| HC35 | Nawanshahar / continuation after 13 Dec / Basmati 1509 | 2.14 | 3000 / 3500 / 3400 | Prices match; arrivals differ from sheet 4.28; full agreement NO |

HC34 and HC35 each have an empty date cell following the dated row within the same market; all cells inspected have no rowspan attribute. The dated anchor row shows the same arrivals as the continuation (34.88 and 2.14 respectively). This supports investigating display allocation/rounding versus exported total; it does not establish a validated arrivals conversion rule. No raw edits.

Next: October 2025, HC36–HC42, seven selected rows. Months without selected sample rows skipped.

## HC36–HC42 — October 2025 inspected

28 October 2026, Codex inspected live October 2025 / Paddy(Basmati) / Punjab after human CAPTCHA. Units tonnes, INR/quintal; agent source inspection, owner sign-off pending; research sheet unchanged.

| Check | Market / date / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC36 | Abohar / 25 Oct / 1121 | 63.00 | 0 / 0 / 2811 | Arrivals differ from 126; min/max displayed zero versus missing in sheet; modal matches; full agreement NO |
| HC37 | Bhikhiwind (Marrimegha) / 13 Oct / 1121 | 120.00 | 2389 / 2389 / 2389 | All match |
| HC38 | Kahnuwan / 02 Oct / 1121 | 6.00 | 3020 / 3125 / 3125 | All match |
| HC39 | Majitha / continuation after 28 Oct / Basmati 1509 | 49.50 | 2800 / 3100 / 3100 | Prices match; arrivals differ from 99; full agreement NO |
| HC40 | Majitha / continuation after 31 Oct / 1121 | 36.00 | 3860 / 3860 / 3860 | Prices match; arrivals differ from 72; full agreement NO |
| HC41 | Samana(Gajewas) / 09 Oct / 1121 | 70.00 | 2389 / 2389 / 2389 | All match |
| HC42 | Shahkot / 18 Oct / 1121 | 1868.70 | 2389 / 2389 / 2389 | All match |

HC39/40 show empty date cells, no rowspan; preceding same-market dated rows show identical arrivals to continuation. HC36 has a following Basmati 1509 row also showing 63 tonnes. No arrivals division adopted; displayed zeros do not become valid min/max observations. MSP-like values are source-confirmed only, not established as genuine basmati market prices. Next November 2025: HC43–HC47, including duplicate-key pair at Bariwala and large Budalada arrivals.

## HC43–HC47 — November 2025 inspected

28 October 2026, Codex live-page inspection after human CAPTCHA; November 2025 / Paddy(Basmati) / Punjab title verified. Agent inspection; owner sign-off pending; research sheet unchanged.

| Check | Market / date / variety | Live arrivals tonnes | Live min / max / modal INR/quintal | Comparison |
|---|---|---:|---|---|
| HC43 | Bariwala / 18 Nov / Basmati 1509, first row | 250.00 | 3125 / 3525 / 3440 | Prices match; arrivals differ from sheet 2970; full agreement NO |
| HC44 | Bariwala / blank date after 18 Nov / Basmati 1509, second row | 2720.00 | 3200 / 3435 / 3350 | Prices match; arrivals differ from sheet 2970; full agreement NO |
| HC45 | Budalada / 08 Nov / Basmati 1509 | 19000.00 | 2890 / 3035 / 2958.68 | All match; source confirmation only, not economic validation |
| HC46 | Kot ise Khan / 11 Nov / 1121 | 1892.00 | 2389 / 2389 / 2389 | All match; commodity/MSP interpretation unresolved |
| HC47 | Nawanshahar / blank date after 11 Nov / Basmati 1509 | 4.45 | 2922 / 3400 / 3200 | Prices match; arrivals differ from sheet 6.405; full agreement NO |

Bariwala source really shows two same-variety rows under the same date context with different prices and arrivals; retain duplicate-key evidence. HC43+HC44 arrivals sum to 2970, matching the exported total, but their unequal split disproves any universal equal division assumption. Nawanshahar dated 1121 anchor shows arrivals 1.96; continuation shows 4.45 (display sum 6.41 versus exported 6.405, consistent with but not proving rounding). Empty date cells have no rowspan. Do not infer an approved arrivals conversion. Next HC48: December 2025 / Bareta / 1121 / 16 December, arrivals 40, min 3100, max 3915, modal 3600.

## HC48 — December 2025 inspected

28 October 2026, Codex live report after human CAPTCHA: December 2025 / Paddy(Basmati) / Punjab title verified. Bareta APMC, 16/12/2025, 1121: arrivals 40.00 tonnes; min 3100, max 3915, modal 3600 INR/quintal. All fields match HC48. Agent source inspection only; owner sign-off pending; research sheet unchanged. Next HC49: January 2026 / Khanna APMC / 1121 / 8 January; sheet arrivals 18239.29 tonnes and all prices 2850.

## HC49 — January 2026 inspected

28 October 2026, Codex live report after human CAPTCHA: title January 2026 / Paddy(Basmati) / Punjab verified. Khanna APMC, 08/01/2026, 1121: arrivals 18,239.29 tonnes; min/max/modal each 2850 INR/quintal. All fields match HC49. Large arrivals are confirmed as displayed by the source, not independently established as economically accurate. Agent inspection only; owner sign-off pending; research sheet unchanged. Next September 2026 HC50–HC52: Bareta duplicate pair on 11 September and Nabha on 19 September.

## HC50–HC52 — September 2026 inspected

28 October 2026, Codex live report after human CAPTCHA: title September 2026 / Paddy(Basmati) / Punjab verified. Agent source inspection only; owner sign-off pending; research sheet unchanged.

| Check | Market / date / variety | Live arrivals tonnes | Live min / max / modal INR/quintal | Comparison |
|---|---|---:|---|---|
| HC50 | Bareta / 11 Sep / 1121, dated row | 40.00 | 3845 / 3875 / 3850 | All fields match |
| HC51 | Bareta / blank date after 11 Sep / 1121 | 25.00 | 3790 / 3835 / 3835 | Numeric fields match; date continuation observed |
| HC52 | Nabha / 19 Sep / Basmati 1509 | 800.00 | 3600 / 4150 / 3900 | All fields match |

Bareta's same-variety duplicate pair is present in the source with distinct arrivals/prices; do not collapse it. HC51 has a separate empty date cell, no rowspan; do not mark merged-cell confirmation yes. Next and final report in this continuation: October 2026 HC53–HC56 (Ajnala and Patran). HC01–HC25 still lack recorded evidence in the saved sheet.

## HC53–HC56 — October 2026 inspected

28 October 2026, Codex live report after human CAPTCHA; October 2026 / Paddy(Basmati) / Punjab title verified. Units tonnes and INR/quintal.

| Check | Market / date context / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC53 | Ajnala / dated 08 Oct / Basmati 1509 | 700 | 3950 / 3950 / 3950 | All fields match |
| HC54 | Patran / blank date after 19 Oct / Basmati 1509 | 500 | 4350 / 4350 / 4350 | Numeric fields match; date continuation observed |
| HC55 | Patran / explicitly dated 21 Oct / Sugandha | 200 | 4590 / 4590 / 4590 | Numeric/date value match, but website has explicit date while saved sheet marks inferred |
| HC56 | Patran / blank date after 21 Oct / 1121 | 150 | 4400 / 4400 / 4400 | Numeric fields match; date continuation observed |

No rowspan attributes in inspected rows. Patran 19 October anchor 1121 has arrivals 23, followed by Sugandha 97 and Basmati 1509 500. Patran 21 October anchor Sugandha has arrivals 200, followed by Basmati 1509 600 and 1121 150. Thus HC55's date-placement representation differs between saved sheet and current page. Preserve this structural difference rather than recording merged-cell confirmation. Agent inspection only; owner sign-off pending; research sheet unchanged.

## Continuation status

HC26–HC56 (31 selected rows) inspected against live AGMARKNET reports in this session. Every observation above is retained, including arrivals, missing-versus-zero, and date-layout discrepancies. This does NOT complete B.2: HC01–HC25 have no saved verification evidence found, the owner's sign-off remains pending, and the research sheet still requires reconciliation in its writable workspace. Continue at October 2021 (HC01–HC05) to cover the undocumented earlier portion. No training or dataset-status promotion authorized by these checks.

## HC01–HC05 — October 2021 inspected

28 October 2026, Codex inspected live October 2021 / Paddy(Basmati) / Punjab after human CAPTCHA. Units tonnes and INR/quintal; agent source inspection only, owner sign-off pending; research sheet unchanged.

| Check | Market / date / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC01 | Kot ise Khan / 31 Oct / 1121 | 1.00 | 700 / 900 / 800 | All match; low price source-confirmed, not proven economically accurate |
| HC02 | Maur / 30 Oct / 1121 | 52.00 | 2990 / 0 / 3410 | Max displayed zero versus sheet missing; other values match; full agreement NO |
| HC03 | Nabha / 19 Oct / Basmati 1509 | 4.50 | 2700 / 3000 / 2900 | All match |
| HC04 | Patran / blank date after 25 Oct / 1121 | 850.00 | 2725 / 3240 / 3025 | Prices match; arrivals differ from sheet 1700; full agreement NO |
| HC05 | Tarantaran / 22 Oct / 1121 | 193.00 | 2700 / 2970 / 2850 | All match |

HC04 follows dated Basmati 1509 row showing arrivals 850.00; empty date cell, no rowspan. No arrivals conversion adopted, no missing max replaced by zero. Next November 2021 HC06–HC08 (Jalalabad, Nabha, Rajpura).

## HC06–HC08 — November 2021 inspected

28 October 2026, Codex live report after human CAPTCHA: November 2021 / Paddy(Basmati) / Punjab title verified. Units tonnes and INR/quintal; agent source inspection only, owner sign-off pending; research sheet unchanged.

| Check | Market / date / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC06 | Jalalabad / 25 Nov / 1121 | 1690.00 | 3525 / 3880 / 3700 | All fields match |
| HC07 | Nabha / blank date after 05 Nov / Basmati 1509 | 50.00 | 2700 / 3100 / 3000 | Prices match; arrivals differ from sheet 100; full agreement NO |
| HC08 | Rajpura / blank date after 29 Nov / 1121 | 428.10 | 3500 / 4000 / 3750 | Prices match; arrivals differ from sheet 856.2; full agreement NO |

HC07 and HC08 have separate empty date cells with no rowspan; dated anchors in the same market show identical arrivals to their continuation (50.00 and 428.10). No arrivals conversion adopted. 39 of 56 sample rows now inspected; 17 remain in eight reports. Next December 2021 HC09, Rajpura / 07 Dec / 1121: sheet arrivals 597, prices 3400 / 4000 / 3700.

## HC09 — December 2021 inspected

28 October 2026, Codex live report after human CAPTCHA: December 2021 / Paddy(Basmati) / Punjab verified. Rajpura APMC: 07/12/2021 Basmati 1509 anchor shows 298.50 tonnes; following blank-date 1121 row shows 298.50 tonnes, min 3400, max 4000, modal 3700 INR/quintal. HC09 prices match; arrivals differ from sheet 597, so full agreement NO. Empty date cell, no rowspan. No arrivals conversion adopted. Agent inspection only; owner sign-off pending; research sheet unchanged. 40/56 inspected, 16 remain across seven reports. Next January 2022 HC10, Malerkotla / 21 Jan / 1121, arrivals 8, prices 5500 / 6500 / 6000.

## HC10 — January 2022 inspected

28 October 2026, Codex live report after human CAPTCHA: title January 2022 / Paddy(Basmati) / Punjab verified. Malerkotla APMC, 21/01/2022, 1121: arrivals 8.00 tonnes; min 5500, max 6500, modal 6000 INR/quintal. All fields match HC10. High price source-confirmed only; economic accuracy not independently validated. Agent inspection only; owner sign-off pending; research sheet unchanged. 41/56 inspected; 15 rows remain in six reports. Next September 2022 HC11, Rayya / 25 Sep / Basmati 1509, arrivals 182, min 2800, max missing, modal 3100.

## HC11 — September 2022 inspected

28 October 2026, Codex live report after human CAPTCHA: title September 2022 / Paddy(Basmati) / Punjab verified. Rayya APMC, 25/09/2022, Basmati 1509: arrivals 182.00 tonnes; min 2800, max 0.00, modal 3100 INR/quintal. Date, arrivals, min and modal match HC11; max is displayed zero versus missing in sheet, so full agreement NO. Do not convert missing maximum into a valid zero price. Agent inspection only; owner sign-off pending; research sheet unchanged. 42/56 inspected; 14 remain across five reports. Next October 2022 HC12–HC16: Bariwala, Dasuya, Patiala(New Anaj Mandi), Patran.

## HC12–HC16 — October 2022 inspected

28 October 2026, Codex live report after human CAPTCHA: October 2022 / Paddy(Basmati) / Punjab verified. Units tonnes and INR/quintal; agent source inspection only, owner sign-off pending; research sheet unchanged.

| Check | Market / date / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC12 | Bariwala / 05 Oct / 1121 | 10 | 2060 / 2060 / 2060 | All match |
| HC13 | Dasuya / 14 Oct / 1121 | 1725 | 2060 / 2060 / 2060 | All match |
| HC14 | Patiala(New Anaj Mandi) / 17 Oct / Basmati 1509 | 30.5 | 3200 / 3450 / 3300 | All match |
| HC15 | Patran / blank date after 19 Oct / Sugandha | 800 | 3280 / 3810 / 3560 | Prices match; arrivals differ from sheet 2400; full agreement NO |
| HC16 | Patran / blank date after 29 Oct / 1121 | 782 | 3160 / 3900 / 3490 | Prices match; arrivals differ from sheet 2346; full agreement NO |

Patran 19 October has three varieties each displaying 800 tonnes; 29 October has three each displaying 782. The sheet carries group totals. No conversion rule adopted; date continuation observed, not merged-cell confirmation. 47/56 inspected; nine rows remain across four reports. Next November 2022 HC17–HC19.

## HC17–HC19 — November 2022 inspected

28 October 2026, Codex live report after human CAPTCHA: November 2022 / Paddy(Basmati) / Punjab verified. Source: https://agmarknet.gov.in/datewisespeccommodityoutput . Units tonnes and INR/quintal. Agent inspection only; owner sign-off pending; research sheet and raw data unchanged.

| Check | Market / date / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC17 | Dasuya / 05 Nov / 1121 | 201 | 2016 / 2016 / 2016 | All match |
| HC18 | Patran / blank date after 17 Nov / 1121 | 100 | 3390 / 4325 / 3870 | Prices match; arrivals differ from sheet 300; full agreement NO |
| HC19 | Patran / blank date after 23 Nov / Sugandha | 200 | 3500 / 3990 / 3790 | Prices match; arrivals differ from sheet 600; full agreement NO |

Patran displays three varieties each with 100 tonnes on 17 November and three each with 200 tonnes on 23 November. Blank date continuation observed; no merged-cell confirmation or conversion rule adopted. 50/56 inspected; six rows remain across three reports: December 2022 HC20, September 2023 HC21–HC22, October 2023 HC23–HC25. Next form: December 2022 / Punjab / Paddy(Basmati), awaiting human CAPTCHA submission.


## HC20 — December 2022 inspected

28 October 2026, Codex live report after human CAPTCHA: December 2022 / Paddy(Basmati) / Punjab verified at https://agmarknet.gov.in/datewisespeccommodityoutput . Malout APMC, 03/12/2022, 1121: arrivals 185.40 metric tonnes; minimum 3600, maximum 4090, modal 3880 INR/quintal. All selected fields match HC20. Agent inspection only; owner sign-off pending; raw data and research sheet unchanged. 51/56 inspected; five rows remain across two reports: September 2023 HC21–HC22 and October 2023 HC23–HC25. Next form prepared: September 2023 / Punjab / Paddy(Basmati), awaiting human CAPTCHA submission.


## HC21–HC22 — September 2023 inspected

28 October 2026, Codex live report after human CAPTCHA: September 2023 / Paddy(Basmati) / Punjab verified at https://agmarknet.gov.in/datewisespeccommodityoutput . Units metric tonnes and INR/quintal.

| Check | Market / date / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC21 | Mehta / 16 Sep / Basmati 1509 | 188 | 3400 / 3650 / 3540 | All match |
| HC22 | Patiala(New Anaj Mandi) / 25 Sep / Basmati 1509 | 23 | 3425 / 3675 / 3530 | All match |

Agent inspection only; owner sign-off pending; raw data and research sheet unchanged. 53/56 inspected; three rows remain in one report: October 2023 HC23–HC25. Next form: October 2023 / Punjab / Paddy(Basmati), awaiting human CAPTCHA submission.


## HC23–HC25 — October 2023 inspected; selected-row inspection complete

28 October 2026, Codex live report after human CAPTCHA: October 2023 / Paddy(Basmati) / Punjab verified at https://agmarknet.gov.in/datewisespeccommodityoutput . Units metric tonnes and INR/quintal.

| Check | Market / date / variety | Live arrivals | Live min / max / modal | Comparison |
|---|---|---:|---|---|
| HC23 | Abohar / blank date after 17 Oct / 1121 | 14.50 | 3900 / 3900 / 3900 | Prices match; arrivals differ from sheet 29; full agreement NO |
| HC24 | Bareta / 24 Oct / 1121 | 150 | 2970 / 3940 / 3640 | All match |
| HC25 | Samana(Kakrala) / blank date after 27 Oct / Basmati 1509 | 285 | 3000 / 3650 / 3300 | Prices match; arrivals differ from sheet 570; full agreement NO |

Abohar shows two varieties each displaying 14.50 tonnes on 17 October; Samana(Kakrala) shows two each displaying 285 on 27 October. Blank date continuation observed; no merged-cell confirmation or arrivals conversion rule adopted. Agent inspection only; raw data and research sheet unchanged.

### Current checkpoint after HC23–HC25

All 56 selected rows now have agent source-inspection observations in this handoff. No monthly CAPTCHA reports remain for this selected sample. This does NOT mark B.2 or the dataset validated: arrivals discrepancies, missing-versus-zero prices and date-layout differences require reconciliation; owner sign-off remains pending. The research hand-check sheet still needs the observations transferred in its writable workspace, preserving actual disagreements and pending human fields. Do not overwrite raw exports, divide arrivals automatically, promote UNKNOWN assumptions, or begin training on the strength of this inspection alone.
