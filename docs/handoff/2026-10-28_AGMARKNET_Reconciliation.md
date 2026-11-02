# AGMARKNET sample reconciliation — 28 October 2026

> **Status added 2 Nov 2026:** historical record. Later on 28 Oct the observations were transferred to the research repository and the owner signed off (research `results/L2_cleaning/hand_check_signoff.json`, `layers/L2_cleaning/CLEANING_EVIDENCE.md` §6, `results/audit/AUDIT_2026-10-28.md`). Do not redo the continuation steps or treat "pending" below as current. Still open: B.2b second-source check; arrivals meaning UNKNOWN.

## Scope and authority

Task ID: RESEARCH-B2-RECON-2026-10-28. Phase: research L2 documentation and contract preparation. Status: agent inspection consolidated; human review and research-workspace transfer pending.

Input: [dated live-source observations](2026-10-28_AGMARKNET_Hand_Check.md) and the unchanged research hand-check sheet. This table transcribes recorded observations; it is not a fresh source retrieval or scientific ground truth. Checked by Codex on 28 October 2026 after user CAPTCHA submission. Owner sign-off is PENDING for every row. Original observations are preserved.

## Reconciled findings

- 56/56 selected rows inspected; 33/56 agree numerically in arrivals and all three prices, counting missing versus displayed zero as disagreement.
- 20 arrivals disagreements; 4 missing-versus-zero disagreements; HC36 belongs to both. Union: 23 rows with numeric discrepancies.
- 56/56 modal prices match the source. Available minimum and maximum prices match; four rows have missing source-export bounds displayed as zero on the live page. Source agreement does not prove prices economically correct or classify unusual prices as errors.
- HC55 additionally differs in date placement: current page explicitly dates its row; export/parser marks it inferred. Date value matches. Blank date continuation and actual merged-cell markup are separate observations.
- Bariwala HC43/HC44 has an unequal 250 + 2720 = 2970 tonne split. Exported group totals cannot be universally divided equally. Rounding also matters (HC34/HC47).
- This is a purposive, stratified sample. Neither its agreement rate nor flags can be used as population accuracy, error precision/recall, or model accuracy.

## Row ledger

Arrivals in metric tonnes; prices INR/quintal. Dates below are the matched date context, not necessarily literal text in the target date cell. `blank` denotes an empty date cell after the same-market dated anchor. `explicit` denotes the current report's literal date. Full-row approval is not inferred from numeric agreement. For exact page-layout evidence consult the dated observations.

| Check | Source row | Date context | Date layout | Export arrivals | Live arrivals | Live min / max / modal | Numeric result |
|---|---|---|---|---:|---:|---|---|
| HC01 | 2021-10_punjab.csv:149 | 2021-10-31 | explicit | 1.0 | 1.0 | 700.0 / 900.0 / 800.0 | numeric match |
| HC02 | 2021-10_punjab.csv:193 | 2021-10-30 | explicit | 52.0 | 52.0 | 2990.0 / 0 / 3410.0 | missing bounds vs zero |
| HC03 | 2021-10_punjab.csv:207 | 2021-10-19 | explicit | 4.5 | 4.5 | 2700.0 / 3000.0 / 2900.0 | numeric match |
| HC04 | 2021-10_punjab.csv:252 | 2021-10-25 | blank | 1700.0 | 850 | 2725.0 / 3240.0 / 3025.0 | arrivals differ |
| HC05 | 2021-10_punjab.csv:347 | 2021-10-22 | explicit | 193.0 | 193.0 | 2700.0 / 2970.0 / 2850.0 | numeric match |
| HC06 | 2021-11_punjab.csv:141 | 2021-11-25 | explicit | 1690.0 | 1690.0 | 3525.0 / 3880.0 / 3700.0 | numeric match |
| HC07 | 2021-11_punjab.csv:215 | 2021-11-05 | blank | 100.0 | 50 | 2700.0 / 3100.0 / 3000.0 | arrivals differ |
| HC08 | 2021-11_punjab.csv:345 | 2021-11-29 | blank | 856.2 | 428.10 | 3500.0 / 4000.0 / 3750.0 | arrivals differ |
| HC09 | 2021-12_punjab.csv:133 | 2021-12-07 | blank | 597.0 | 298.50 | 3400.0 / 4000.0 / 3700.0 | arrivals differ |
| HC10 | 2022-01_punjab.csv:14 | 2022-01-21 | explicit | 8.0 | 8.0 | 5500.0 / 6500.0 / 6000.0 | numeric match |
| HC11 | 2022-09_punjab.csv:60 | 2022-09-25 | explicit | 182.0 | 182.0 | 2800.0 / 0 / 3100.0 | missing bounds vs zero |
| HC12 | 2022-10_punjab.csv:13 | 2022-10-05 | explicit | 10.0 | 10.0 | 2060.0 / 2060.0 / 2060.0 | numeric match |
| HC13 | 2022-10_punjab.csv:46 | 2022-10-14 | explicit | 1725.0 | 1725.0 | 2060.0 / 2060.0 / 2060.0 | numeric match |
| HC14 | 2022-10_punjab.csv:176 | 2022-10-17 | explicit | 30.5 | 30.5 | 3200.0 / 3450.0 / 3300.0 | numeric match |
| HC15 | 2022-10_punjab.csv:200 | 2022-10-19 | blank | 2400.0 | 800 | 3280.0 / 3810.0 / 3560.0 | arrivals differ |
| HC16 | 2022-10_punjab.csv:224 | 2022-10-29 | blank | 2346.0 | 782 | 3160.0 / 3900.0 / 3490.0 | arrivals differ |
| HC17 | 2022-11_punjab.csv:61 | 2022-11-05 | explicit | 201.0 | 201.0 | 2016.0 / 2016.0 / 2016.0 | numeric match |
| HC18 | 2022-11_punjab.csv:259 | 2022-11-17 | blank | 300.0 | 100 | 3390.0 / 4325.0 / 3870.0 | arrivals differ |
| HC19 | 2022-11_punjab.csv:274 | 2022-11-23 | blank | 600.0 | 200 | 3500.0 / 3990.0 / 3790.0 | arrivals differ |
| HC20 | 2022-12_punjab.csv:54 | 2022-12-03 | explicit | 185.4 | 185.4 | 3600.0 / 4090.0 / 3880.0 | numeric match |
| HC21 | 2023-09_punjab.csv:63 | 2023-09-16 | explicit | 188.0 | 188.0 | 3400.0 / 3650.0 / 3540.0 | numeric match |
| HC22 | 2023-09_punjab.csv:78 | 2023-09-25 | explicit | 23.0 | 23.0 | 3425.0 / 3675.0 / 3530.0 | numeric match |
| HC23 | 2023-10_punjab.csv:15 | 2023-10-17 | blank | 29.0 | 14.50 | 3900.0 / 3900.0 / 3900.0 | arrivals differ |
| HC24 | 2023-10_punjab.csv:70 | 2023-10-24 | explicit | 150.0 | 150.0 | 2970.0 / 3940.0 / 3640.0 | numeric match |
| HC25 | 2023-10_punjab.csv:478 | 2023-10-27 | blank | 570.0 | 285 | 3000.0 / 3650.0 / 3300.0 | arrivals differ |
| HC26 | 2023-11_punjab.csv:25 | 2023-11-14 | blank | 700.0 | 350 | 3000.0 / 3800.0 / 3600.0 | arrivals differ |
| HC27 | 2023-12_punjab.csv:39 | 2023-12-09 | explicit | 100.0 | 100.0 | 3800.0 / 5000.0 / 4800.0 | numeric match |
| HC28 | 2024-09_punjab.csv:45 | 2024-09-23 | explicit | 103.0 | 103.0 | 2250.0 / 0 / 2646.0 | missing bounds vs zero |
| HC29 | 2024-10_punjab.csv:191 | 2024-10-20 | explicit | 3560.0 | 3560.0 | 2320.0 / 2320.0 / 2320.0 | numeric match |
| HC30 | 2024-11_punjab.csv:527 | 2024-11-18 | blank | 42.0 | 21 | 2600.0 / 3200.0 / 2800.0 | arrivals differ |
| HC31 | 2024-12_punjab.csv:16 | 2024-12-01 | explicit | 24.0 | 24.0 | 4100.0 / 4300.0 / 4200.0 | numeric match |
| HC32 | 2024-12_punjab.csv:64 | 2024-12-12 | explicit | 2000.0 | 2000.0 | 3400.0 / 3925.0 / 3500.0 | numeric match |
| HC33 | 2024-12_punjab.csv:149 | 2024-12-09 | explicit | 146.0 | 146.0 | 2550.0 / 4330.0 / 3200.0 | numeric match |
| HC34 | 2024-12_punjab.csv:221 | 2024-12-04 | blank | 69.75 | 34.88 | 3475.0 / 4221.0 / 3700.0 | arrivals differ |
| HC35 | 2024-12_punjab.csv:235 | 2024-12-13 | blank | 4.28 | 2.14 | 3000.0 / 3500.0 / 3400.0 | arrivals differ |
| HC36 | 2025-10_punjab.csv:20 | 2025-10-25 | explicit | 126.0 | 63 | 0 / 0 / 2811.0 | arrivals differ; missing bounds vs zero |
| HC37 | 2025-10_punjab.csv:58 | 2025-10-13 | explicit | 120.0 | 120.0 | 2389.0 / 2389.0 / 2389.0 | numeric match |
| HC38 | 2025-10_punjab.csv:180 | 2025-10-02 | explicit | 6.0 | 6.0 | 3020.0 / 3125.0 / 3125.0 | numeric match |
| HC39 | 2025-10_punjab.csv:247 | 2025-10-28 | blank | 99.0 | 49.50 | 2800.0 / 3100.0 / 3100.0 | arrivals differ |
| HC40 | 2025-10_punjab.csv:253 | 2025-10-31 | blank | 72.0 | 36 | 3860.0 / 3860.0 / 3860.0 | arrivals differ |
| HC41 | 2025-10_punjab.csv:419 | 2025-10-09 | explicit | 70.0 | 70.0 | 2389.0 / 2389.0 / 2389.0 | numeric match |
| HC42 | 2025-10_punjab.csv:433 | 2025-10-18 | explicit | 1868.7 | 1868.7 | 2389.0 / 2389.0 / 2389.0 | numeric match |
| HC43 | 2025-11_punjab.csv:46 | 2025-11-18 | explicit | 2970.0 | 250 | 3125.0 / 3525.0 / 3440.0 | arrivals differ |
| HC44 | 2025-11_punjab.csv:47 | 2025-11-18 | blank | 2970.0 | 2720 | 3200.0 / 3435.0 / 3350.0 | arrivals differ |
| HC45 | 2025-11_punjab.csv:97 | 2025-11-08 | explicit | 19000.0 | 19000.0 | 2890.0 / 3035.0 / 2958.68 | numeric match |
| HC46 | 2025-11_punjab.csv:167 | 2025-11-11 | explicit | 1892.0 | 1892.0 | 2389.0 / 2389.0 / 2389.0 | numeric match |
| HC47 | 2025-11_punjab.csv:277 | 2025-11-11 | blank | 6.405 | 4.45 | 2922.0 / 3400.0 / 3200.0 | arrivals differ |
| HC48 | 2025-12_punjab.csv:14 | 2025-12-16 | explicit | 40.0 | 40.0 | 3100.0 / 3915.0 / 3600.0 | numeric match |
| HC49 | 2026-01_punjab.csv:21 | 2026-01-08 | explicit | 18239.29 | 18239.29 | 2850.0 / 2850.0 / 2850.0 | numeric match |
| HC50 | 2026-09_punjab.csv:76 | 2026-09-11 | explicit | 40.0 | 40.0 | 3845.0 / 3875.0 / 3850.0 | numeric match |
| HC51 | 2026-09_punjab.csv:77 | 2026-09-11 | blank | 25.0 | 25.0 | 3790.0 / 3835.0 / 3835.0 | numeric match |
| HC52 | 2026-09_punjab.csv:230 | 2026-09-19 | explicit | 800.0 | 800.0 | 3600.0 / 4150.0 / 3900.0 | numeric match |
| HC53 | 2026-10_punjab.csv:23 | 2026-10-08 | explicit | 700.0 | 700.0 | 3950.0 / 3950.0 / 3950.0 | numeric match |
| HC54 | 2026-10_punjab.csv:237 | 2026-10-19 | blank | 500.0 | 500.0 | 4350.0 / 4350.0 / 4350.0 | numeric match |
| HC55 | 2026-10_punjab.csv:241 | 2026-10-21 | explicit | 200.0 | 200.0 | 4590.0 / 4590.0 / 4590.0 | numeric match |
| HC56 | 2026-10_punjab.csv:242 | 2026-10-21 | blank | 150.0 | 150.0 | 4400.0 / 4400.0 / 4400.0 | numeric match |

## Rule reconciliation

1. **CONFIRMED observation:** sampled blank-date entries are visible beneath same-market date anchors. The proposed explanation that this proves a merged HTML date cell is contradicted by the recorded no-rowspan observations. Do not mark merged-cell confirmation YES. Date carry-forward remains a traceable interpretation, not validation of every one of 580 rows.
2. **UNKNOWN:** exact historical export arrivals semantics and any general recovery of per-variety arrivals. Preserve exported arrivals with provenance; preserve web arrivals as separate observations. Do not replace totals, assign equal shares or use these ambiguous values as variety-level weights.
3. **DESIGN DECISION retained:** missing remains missing; live zero bounds are evidence of representation differences and violate positive-price ordering. Do not impute zero as a valid price bound.
4. **DESIGN DECISION retained:** duplicate market/date/variety rows with distinct values remain distinct and flagged. Do not average, deduplicate or declare erroneous because keys repeat.
5. **ENGINEERING LEAN:** retain both dated-only and inferred-date unweighted median index variants for sensitivity analysis. Existing index code uses modal-price medians, so no arrivals correction is required to preserve that computation; inferred-date inclusion and duplicate handling still require review. No primary series selected here.

## Next research-workspace task contract

Task ID: RESEARCH-B2-TRANSFER-v1. Status: PREPARED, not executed. Inputs: unchanged 56-row sheet plus this ledger and dated observations. Output: reviewed machine-readable agent-observation ledger, updated sheet, cleaning evidence and research status; scripts must generate research results. Preserve original file/line identifiers and hashes. Keep agent checker separate from human reviewer and leave human sign-off pending. Do not report B.2 complete merely because observations were transferred.

- Transfer exact web values and date layout; preserve raw data and original exports. `web_date_shown` must describe blank versus explicit text rather than silently substituting the inferred date. Where the legacy merged-cell column conflates placement with markup, record NO for documented no-rowspan checks and UNKNOWN otherwise; add a separate date-context/layout field instead of fabricating YES.
- Add separate arrival agreement, bounds agreement, modal agreement and representation discrepancy fields. A numeric match is not owner approval or independent source verification.
- Recompute the ledger counts: 56 unique IDs, 20 arrivals disagreements, 4 missing/zero disagreements, 23 in their union, 33 numeric matches, 56 modal matches. Confirm each observation against its note; fail explicitly on missing/duplicate IDs or absent provenance.
- Record raw-file SHA256 before and after. Run existing parser/index/release tests from the pinned research environment; report actual commands and versions. Do not claim these tests passed from documentation review.
- Amend cleaning evidence and instructions to remove the unsupported implication that date continuation proves merged cells. Keep historical plans intact and record the correction as a dated amendment.
- B.2b remains open: obtain comparable independent market/date/variety evidence. Press ranges and AGMARKNET republishers cannot supply exact per-row ground truth. Record unavailable cells rather than substitute another commodity/date.
- B.3 depends on reviewed labels and an approved evaluation protocol. Source equality alone cannot label a price genuine/error. Do not compute error precision/recall from it, choose MAD/Hampel/IQR thresholds, select a winner, or start forecasting without resolving the recorded prerequisites.

Scope excludes production code, database writes, provider messages, raw edits, numerical policy changes, release promotion and training. No product gate is lifted. No concurrency or database effects. Idempotency: identical inputs must reproduce identical ledger rows; access/check dates remain evidence metadata, not regenerated assertions.

## Independent-source follow-up

Accessed 28 October 2026. eNAM's [historical dashboard](https://www.enam.gov.in/web/dashboard/Historical) was discovered, but direct retrieval timed out; no historical comparison extracted and no claim that the service is universally unavailable. The Tribune's [18 October 2025 report](https://www.tribuneindia.com/news/punjab/basmati-growers-face-losses-as-prices-crash-by-rs1500-quintal/amp/) was read: regional 1509 ranges and a Fazilka trader's range provide context, without a matched day/market/modal series for the preselected 15 cells. This is not a completed independent spot check. No tolerance was adopted retrospectively and no independent agreement metric computed.

## Completion evidence and remaining dependency

Completed here: consolidated all 56 documented observations, computed discrepancy totals, recorded the cleaning-rule conflict, updated the planning tracker and prepared the bounded transfer contract. Verification: unique sample IDs and override sets checked by Python; ledger count checked after writing. These are documentation checks only.

Research CSV, parser, manifests, research STATUS and raw files were not changed. The research sibling is outside this session's writable roots; open `/Users/arijitsingh/Documents/kisaniti_research_paper` as the workspace to execute the transfer and pipeline task. Human sign-off, independent-source comparison, cleaning experiments and final dataset validation remain incomplete.

Input sheet SHA256: `a7e8d80a50bb3ffe78b39add69ac7431a83b15aeadfb5b157305374dc4da5e37`.
