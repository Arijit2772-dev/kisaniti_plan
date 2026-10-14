# Patiala web research: storage-risk approximation and likely answers to the 21 unknowns

15 October 2026 • Desk research requested by the user (based in Patiala, Punjab) • **ENGINEERING LEAN / DESK-RESEARCH APPROXIMATION** — not FPO-interview evidence, not field validation.

The user asked for (1) an approximate spoilage-risk method for paddy and wheat using only the automatic ESP32 + SHT30 temperature/humidity data, and (2) the most likely answers to the 21 open real-world questions (CORE-001…009, PILOT-001…012) for Patiala. This document gives those approximations with sources. It does **not** close any `UNKNOWN`: each answer is a demo assumption plus the single cheapest check that would confirm it. Registers link here; the unknown register status stays `OPEN`.

---

## Part A — Important finding first: MSP flattens mandi price differences for paddy and wheat

**Evidence.** In Punjab, ordinary (non-basmati) paddy and wheat are bought by government agencies at the Minimum Support Price (MSP) through arhtiyas in notified mandis/purchase centres:

- MSP KMS 2025-26 paddy: ₹2,369/q common, ₹2,389/q Grade A ([DES MSP statement](https://desagri.gov.in/wp-content/uploads/2025/12/MSP-Statement_English_As-on-12.12.2025.pdf)).
- MSP RMS 2026-27 wheat: ₹2,585/q ([Tribune](https://www.tribuneindia.com/news/india/govt-raises-wheat-msp-by-6-59-to-rs-2585-per-quintal-for-2026-27)). Punjab mandi wheat averaged ₹2,597/q in May 2026, i.e. within ~0.5% of MSP ([UNI](https://test.uniindia.com/news/north/wheat-fetched-above-msp/3926076.html)).
- Patiala district alone runs ~109 mandis plus ~126 temporary yards in paddy season ([Tribune, Patiala](https://www.tribuneindia.com/news/punjab/57-officials-including-16-pcs-officers-to-ensure-seamless-procurement-in-patiala/amp)); the state notifies ~1,850 purchase centres ([Tribune](https://www.tribuneindia.com/news/punjab/smooth-purchase-on-mind-govt-back-to-offline-passes-438773)).
- Basmati is **not** MSP-procured and is sold to private traders, so its price varies by mandi and day ([Tribune, Amritsar](https://www.tribuneindia.com/news/amritsar/kisan-union-protests-seeks-higher-msp-for-basmati-yield)). Aggregated AGMARKNET-derived pages show e.g. Rajpura ₹3,750/q vs Patran ₹3,600/q in September 2026 ([acrop Rajpura](https://acrop.app/mandi/punjab/patiala/rajpura/paddybasmati), [acrop Patran](https://acrop.app/mandi/punjab/patiala/patran/paddybasmati)); treat these as indicative only.

**Implication (recorded as a CONFLICT, not silently applied).** For MSP paddy/wheat in Patiala, *which mandi* barely changes the price; the system's value there is mainly in **timing, moisture compliance, storage risk, congestion and the nearest/cheapest trip**. Price forecasting and mandi choice matter most for (a) **basmati paddy** and (b) **wheat held back on the farm for later private sale** (farmers did this in 2022 when open-market prices rose above MSP — [Hindustan Times, Patiala](https://www.pressreader.com/india/hindustan-times-patiala/20220520/281668258584488)).

**Recommended demo framing:** keep the architecture unchanged; model the MSP as a known price for MSP-procured series (a labelled "MSP_SCHEDULE" price source, distinct from a model forecast) and use the forecasting candidates where prices actually move (basmati, off-season wheat). See [conflict register](../blueprint/04_CONFLICT_REGISTER.md) entry `PATIALA-MSP-2026-10-15`.

---

## Part B — Storage-risk approximation for paddy and wheat (proposed `StoragePolicy` DEMO v0)

### B1. The key idea: put the probe *inside* the grain and read water activity directly

Grain constantly exchanges moisture with the air trapped between kernels. Once it settles, the relative humidity of that **inter-granular air equals the grain's water activity** (`aw = RH/100`). This is how commercial in-bin and in-bag moisture cables work: they measure inter-granular RH and temperature ([USDA-ARS](https://www.ars.usda.gov/ARSUserFiles/30200525/416MonitoringstoredgrainwrelhumiditytempCO2.pdf); [probe-type bag meter, U. Kentucky](https://uknowledge.uky.edu/bae_facpub/214); [Brazilian silo validation](https://scielo.br/j/rbeaa/a/bX3G5QhKWJKtx8NTFpqW8sP)).

Why this suits KisanNiti:

- Mould limits are published as **water activity / relative humidity thresholds**, which are nearly the **same for paddy and wheat**. We do not need crop-specific moisture-conversion equations to estimate mould risk.
- Converting to "% moisture" needs crop isotherm constants (ASABE D245.6). I could not obtain a verifiable copy during this research, so **no moisture % is displayed**. Re-add it only after the constants are copied from the standard itself.
- A probe hanging in shed air measures **ambient** air, not grain. Ambient placement must give `UNKNOWN` for grain risk (shown as "ambient conditions" only).

### B2. Published thresholds used

| Fact | Source |
|---|---|
| Storage fungi need ≥ 65% RH (`aw` 0.65), ≈ 13% equilibrium moisture in cereals; growth range about 10–40 °C | [FAO, micro-organisms in storage](https://www.fao.org/4/x5065e/x5065E0c.htm) |
| Grain kept in equilibrium with air below 65–70% RH slows or blocks most degradation; safe long-term moisture in hot regions: paddy 14%, wheat 13% | [FAO, grain storage techniques](https://www.fao.org/4/t0522e/T0522E09.htm) |
| Minimum grain moisture for fungi: *A. restrictus* 13.5%, *A. glaucus* 14%, *A. candidus/ochraceus* 15%, *Penicillium* 16.5–19%, *A. flavus* 18% | [FAO](https://www.fao.org/4/x5065e/x5065E0c.htm) |
| Xerophilic *Aspergillus* (e.g. *A. penicillioides*) can grow on grain at `aw` ≈ 0.68 | [mould reference](https://library.bustmold.com/aspergillus/aspergillus-penicillioides/) (secondary) |
| Insects breed roughly 15–35/42 °C, optimum 25–32 °C; little development below ~14 °C | [FAO, storage pests](https://www.fao.org/4/x5065e/x5065E0d.htm); [FAO](https://www.fao.org/4/t0522e/T0522E09.htm) |
| Higher temperature requires drier grain for the same safety | [FAO](https://www.fao.org/4/t0522e/T0522E09.htm); [Canadian Grain Commission](https://www.canada.ca/en/grain-commission/services/grain-quality/manage-stored-grain/prevent-spoilage-stored-grain.html) |
| Main Punjab stored-wheat insects: rice weevil, lesser grain borer, khapra beetle | [Indian J. Agric. Research / PAU](https://arccjournals.com/journal/indian-journal-of-agricultural-research/A-5695) |
| Patiala climate: afternoon RH ~71–75% in Aug–Sep, ~59% Oct–Nov, ~32–33% Apr–May; highs ~35–39 °C in Apr–May | [IMD normals via Wikipedia](https://encycloreader.org/r/wikipedia.php?q=Patiala) |

### B3. Proposed rule (DEMO, experimental; identical for paddy and wheat)

**Inputs:** in-grain temperature `T` (°C) and RH (%) from `SensorReadingV1`; a device assignment with `placement = IN_GRAIN`; crop ∈ {paddy, wheat}.

**Pre-conditions (any failure gives `UNKNOWN` with a reason):**

1. Placement is registered as `IN_GRAIN` (probe tip at mid-depth of a bag/heap, at least ~30 cm from the bag surface where possible). `AMBIENT` gives `UNKNOWN` for grain risk.
2. Settling period: ignore the first **6 h** after insertion or reassignment (proposed; inter-granular RH needs time to equilibrate — measure the real settling time on the bench).
3. Freshness: latest trusted sample ≤ 15 min old (existing ENGINEERING LEAN).
4. Coverage: ≥ 75% of the expected samples in the 24 h window (≥ 216 of 288).
5. Values valid: RH 0–100%, `T` −10 to 60 °C, measurement time trusted.

**Computation:** take the 24 h rolling median of RH and of `T`, so short spikes such as opening the shed or a rain gust do not flip the category. `aw = RH_median / 100`.

**Mould axis (primary):**

| `aw` (24 h median) | Category | Meaning in plain words |
|---|---|---|
| < 0.65 | LOW | Below the FAO storage-fungi minimum; grain is roughly at or below safe storage moisture |
| 0.65 – < 0.70 | MEDIUM | FAO "safeguard" band; xerophilic moulds possible over weeks |
| ≥ 0.70 | HIGH | Storage moulds can grow; freshly harvested paddy (17–22% moisture) lands here immediately |

**Temperature adjustments:**

- If `T_median` < 10 °C, lower the mould category by one level (below FAO's fungal growth range). This rarely applies in Patiala outside Dec–Jan.
- If `T_median` ≥ 25 °C **and** `aw` ≥ 0.60, raise LOW to MEDIUM. Warm grain needs to be drier for the same safety, and insect breeding is at its optimum.

**Insect flag (advisory, shown separately, does not set urgency on its own):** `T_median` ≥ 25 °C → "insect-favourable temperature"; 15–25 °C → "insects slow"; < 15 °C → "insects largely inactive". Temperature cannot detect an actual infestation, so this flag recommends inspection/fumigation checks rather than changing selling priority. This avoids marking every Patiala lot HIGH all summer.

**Hysteresis:** escalate as soon as the 24 h median crosses a boundary. De-escalate only after the median stays at least **2 RH points below** the boundary for **48 h**, so the category does not flicker around 65%.

**Optional later rule (needs a new data source, not selected):** compare in-grain `T` with outdoor Patiala air temperature (e.g. a weather API). If grain warms > 3 °C over 7 days while outdoor air does not, that suggests biological self-heating → HIGH. With only one probe, this is the only way to spot heating.

**Output:** `StorageAssessmentV1` with category, `aw`, `T`, insect flag, window, policy version `STORAGE-DEMO-0.1`, evidence links to this document, and the label "experimental approximation — not measured spoilage".

### B4. What this does and does not claim

- It **estimates conditions that favour mould/insects** at one probe point. It does not detect spoilage, measure moisture %, or predict shelf life. It also cannot speak for other bags or other farms.
- It works the same for wheat and paddy because the fungal thresholds are crop-independent water-activity limits. Crop differences matter only for the moisture % shown on screen, which is deliberately left out.

**Expected Patiala behaviour:**

- Wheat stored from April will usually read LOW/MEDIUM in the dry, hot Apr–Jun months (insect flag on).
- From July–September, jute bags absorb monsoon humidity, so the reading may rise to MEDIUM/HIGH. This matches the known monsoon storage problem in Punjab.
- Freshly harvested paddy starts HIGH, which correctly signals that it must be dried or sold quickly.

### B5. Cheap validation (one-time, no recurring manual readings)

1. **Salt check:** seal the probe for 24 h over saturated table-salt solution; it should read ~75% RH. A hygrometer extension guide uses ~70–74% as the check value ([Purdue PICS](https://ag.purdue.edu/department/entm/extension/pics-network/_docs/hygrometer-revised-9.20.pdf)). Record the offset.
2. **One-time moisture comparison:** take a grain sample from the probed bag to a mandi or agency moisture meter (all procurement mandis test moisture). Compare it with the reading: about 13% wheat / 14% paddy should sit near `aw` 0.65–0.70.
3. **Bench settling test:** insert the probe into a known bag and record how long RH takes to stabilise; replace the 6 h assumption with the measured value.

Until these are recorded, every displayed category carries the experimental label (acceptance test T22 stays required).

---

## Part C — Likely answers to the 21 unknowns for Patiala

Confidence: **H** = several consistent official/news sources; **M** = consistent indirect evidence; **L** = reasoned guess. "Demo assumption" is what we would configure for the college demo (DEMO scope only). "Confirm with" is the cheapest real check, usually one conversation with an FPO CEO, an arhtiya, or the mandi office in Patiala/Rajpura/Samana/Nabha/Patran.

### Core business blockers

| ID | Likely answer in Patiala | Conf. | Demo assumption | Confirm with |
|---|---|---|---|---|
| CORE-001 Where produce stays before sale | Non-basmati paddy: combine-harvested and taken **from field to mandi the same day**; almost never stored on the farm. Wheat: mostly field → mandi in April; a part is kept at home/farm for family use or to **hold for a better private price**. Basmati: sometimes held on the farm. | M | Paddy (MSP) lots: placement `FIELD_TO_MANDI`, no storage monitoring. Wheat & basmati lots: `FARM_SHED` placement with the sensor. | Ask an FPO which crops members actually store and for how long. |
| CORE-002 Execution path | **Farmer → nearest mandi/purchase centre** (Patiala ~109 mandis + temporary yards, so usually a short trip). Mandi → warehouse/miller is the procurement agency's job, not the farmer's or FPO's. | H | Origin = farmer's farm/field; destination = candidate mandi; nothing after the mandi is modelled. | Mandi office / FPO. |
| CORE-003 Who arranges transport | **The farmer**, usually their own or a neighbour's **tractor-trolley**. Agency contractors lift grain from the mandi afterwards ([HT on tractor-trailers](https://www.pressreader.com/india/hindustan-times-amritsar/20180426/281749859964622)). | H | Arranger = FARMER; FPO only advises. | FPO. |
| CORE-004 Transport pricing | Own tractor: diesel + time, rarely recorded. Hired trolley: a **per-trip** local rate. No published Patiala rate found; Mandi Board rate fixing exists only for arhtiya/agency transport ([HT](https://www.pressreader.com/india/hindustan-times-patiala/20180404/281625305867134)). | L | Fixed charge per trolley trip plus a per-km component, both as **labelled synthetic** numbers; one trip carries up to one trolley load; dedicated trips only. | Ask 2–3 farmers what a hired trolley trip to Rajpura/Samana costs. |
| CORE-005 Economic acceptability | MSP crops: any mandi paying **MSP** (₹2,369/₹2,389 paddy, ₹2,585 wheat) is acceptable; farmers prefer the nearest. Basmati/held wheat: a mandi is acceptable only if its net price (after transport) beats the nearest mandi's net price. | M | Choice admissible if expected net ≥ MSP-equivalent (MSP crops) or ≥ nearest-mandi net (non-MSP). | FPO CEO with 3–4 example choices. |
| CORE-006 Smallest scheduling unit | Produce moves by **trolley load** and is packed in **bags** at the mandi. Common bag sizes (often quoted as 50 kg wheat, 37.5 kg paddy) could not be verified this session. | L | Increment **1 quintal (100 kg)** for the demo; increment type kept configurable (bag/quintal/trolley). | Arhtiya: bag weight this season and typical trolley load. |
| CORE-007 Quality/variety | **Yes, it matters.** Paddy: moisture ≤ 17%, foreign matter 1% + 1%, damaged/discoloured ≤ 5%, immature ≤ 3%, admixture ≤ 6%; Grade A gets ₹20/q more ([FCI/state specs](https://epaddyarchive.wb.gov.in/ProcurementArchive2021/Notification/K2021%20paddy%20purchase%20unifrom%20specifications.pdf)). Wheat: moisture ≤ 12%, relaxed after rain damage ([RuralVoice](https://eng.ruralvoice.in/tag/msp-wheat-procurement)). Basmati variety (1121/1509/1718) strongly affects price. | H | Variety is part of the price series ID; moisture over the limit gives a "may be refused at gate" warning; grade is a lot attribute. | Already well documented; confirm current-season relaxations. |
| CORE-008 Mandi load limits | Peak-season **congestion and lifting delays** are real (gate passes, temporary yards, district officials posted to mandis), but there is **no published daily capacity** per mandi. | M | MarketLoadPolicy **ADVISORY** (warning only) using historical arrivals; never STRICT. | Mandi secretary. |
| CORE-009 Off-plan sale vs reservation | Off-plan sales are **likely common**: farmers have long credit ties with their own arhtiya (~48,000 arhtiyas serving ~18.5 lakh farmers — [Business Standard](https://www.pressreader.com/india/business-standard/20201008/281818581293354)). The **digital J-form** proves the actual sale. | M | Off-plan sale entered with its J-form becomes a pending report; manager explicitly releases the conflicting reservation (Layer 5 recommendation). | FPO: how often members sell through their own arhtiya. |

### Must validate before pilot

| ID | Likely answer in Patiala | Conf. | Demo assumption | Confirm with |
|---|---|---|---|---|
| PILOT-001 Produce registration | Farmers already have Aadhaar-linked bank accounts and land records registered on the state procurement portal for direct payment ([Rajya Sabha reply](https://rsdebate.nic.in/bitstream/123456789/751779/1/PQ_266_06122024_U1312_p184_p186.pdf)). The FPO would register the member, crop, variety, expected quantity and storage place. | M | FPO staff enter lots; quantity marked **"declared estimate"**. | FPO. |
| PILOT-002 Who verifies quantity/quality | Pre-sale quantity is the farmer's estimate. The **real weight and moisture are measured at the mandi** (scales, agency moisture meter). | M-H | Lot quantity before sale = estimate; the J-form quantity is the authoritative sold quantity. | Mandi visit. |
| PILOT-003 Staff roles | A typical FPO has a board of directors, a CEO/manager and an accountant (general knowledge of FPO support schemes; not re-checked this session). | L-M | Roles: CEO = approver; accountant = sale recorder; field staff = registration; board = read-only. | FPO. |
| PILOT-004 Farmer acceptance | Farmers own the grain and can't be forced. Acceptance is needed in practice; a WhatsApp reply is the natural channel. | M | Instruction asks "Yes/No"; staff record acceptance as a **separate fact** (not a sale, not a stock change). | FPO. |
| PILOT-005 Who records the actual sale | The **arhtiya and buyer confirm the sale in the Mandi Board system**, which issues a **digital J-form** (QR code, unique number) to the farmer's WhatsApp; also downloadable from emandikaran-pb.in / DigiLocker ([Tribune](https://www.tribuneindia.com/news/punjab/digital-sale-receipts-for-farmers-in-mandis-382488)). Initially this covered MSP paddy and wheat. Payment goes directly to the farmer's bank, typically promised within ~48 h. | H | FPO staff record sales from the J-form; **J-form number = evidence ID and duplicate-check key**; later, an import adapter if access is allowed. | Already well documented; check basmati/private J-form coverage. |
| PILOT-006 Transport expense recorded? | Usually **not** (own tractor). | M | Optional field; missing stays missing. | FPO. |
| PILOT-007 Storage thresholds | See **Part B** (`aw` 0.65 / 0.70, 10 °C and 25 °C adjustments, 24 h median, hysteresis). | M (science) / L (local calibration) | `STORAGE-DEMO-0.1`, labelled experimental. | Part B5 validation. |
| PILOT-008 IoT deployment | One probe **inside** a representative wheat/basmati bag in the farm shed; electronics outside; hotspot Wi-Fi; 5-min samples; MQTT QoS 1 with de-duplication (existing recommendation). | M | As stated; ambient placement allowed but gives `UNKNOWN` for grain risk. | Bench test. |
| PILOT-009 Fairness | No Punjab-specific evidence found. With MSP, the fairness question is mostly **who gets the early/near/uncongested slots**; per-farmer leximin fulfilment is a sensible default. | L | Keep per-farmer leximin. | Show the FPO two example allocations. |
| PILOT-010 Storage vs economics | MSP crops: the price is the same everywhere, so **selling at-risk stock early costs nearly nothing**; storage priority is safe. Basmati/held wheat: needs a limit on how much money urgency may sacrifice. | M (MSP) / L (basmati) | MSP crops: current hierarchy. Non-MSP: storage urgency may not sacrifice more than a demo-configured share of expected net (value to be set and labelled DEMO). | FPO CEO. |
| PILOT-011 Freshness warning/block | Mandi prices publish daily; MSP is fixed for the season; quotes are day-specific; sensors every 5 min. | M | Warn if the forecast's data is older than 1 completed business day; MSP price valid for the notified season; block on expired quote; sensor stale after 15 min → `UNKNOWN`. | Measure actual AGMARKNET publication lag. |
| PILOT-012 Same-day cutoffs | Combine harvesting of paddy is banned at night by district orders (roughly 6–7 pm to 9–10 am, varies by district/year) to keep moisture down ([Barnala order](https://barnala.gov.in/document/complete-ban-on-paddy-harvesting-with-combine-harvesters-from-7-pm-to-8-am/), [Tribune Ludhiana](https://www.tribuneindia.com/news/ludhiana/dc-jain-reviews-arrangements-for-paddy-procurement)). Procurement windows: paddy from ~15 Sep/1 Oct to late Nov; wheat from 1 Apr to ~end May ([Tribune](https://www.tribuneindia.com/news/punjab/procurement-of-paddy-from-sept-15-as-govt-advances-date); [Tribune](https://m.tribuneindia.com/news/punjab/now-232-mandis-open-till-may-31-in-punjab-395149)). | M | No same-day dispatch; approval by 18:00 IST for next-day trips; MSP-crop dates must fall inside the notified procurement window. | Patiala DC's current-season order. |

---

## Part D — What to do with this

1. ~~Decide the demo crop focus.~~ **Decided by the user (CHAT-20):** demo forecasting/mandi choice uses **held wheat + basmati paddy**; MSP paddy/wheat use the fixed MSP_SCHEDULE price path. Conflict `PATIALA-MSP-2026-10-15` is resolved for demo scope.
2. Freeze `STORAGE-DEMO-0.1` (Part B3) in the contract-freeze task (W01) as an experimental DEMO policy, then run B5.
3. Use the Part C "Demo assumption" column to fill the DEMO policy bundle in [continuation tasks](../blueprint/08_CONTINUATION_TASKS.md). Keep each one labelled DEMO.
4. A single 30-minute conversation with one Patiala FPO CEO plus one arhtiya would settle most "L" and "M" rows. Ask in plain words, per architecture §66.

## Limits of this research

- Web searches on 15 Oct 2026; several news items are undated or from earlier seasons. Figures such as bag weights, trolley capacities and transport rates were **not found** and are not invented here.
- The ASABE isotherm constants could not be retrieved; therefore no grain-moisture % is computed.
- No FPO, arhtiya, farmer or mandi official was contacted. Nothing here is a real-FPO validation; all 21 unknowns remain `OPEN` in the register.
