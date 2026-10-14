# Design choices and their alternatives

15 October 2026 • For the research paper and for any agent that wants to change a choice. Each row says what was chosen, what else was considered, why, and **when the alternative would be better**. "Evidence status" says whether the choice is backed by a run, by sources, or only by reasoning. Nothing here is a measured result yet.

Labels: **D** = DESIGN DECISION, **EL** = ENGINEERING LEAN (from the registers). An experiment ID means the choice will be tested.

## System and software

| Area | Chosen | Alternatives | Why chosen | Alternative wins when | Evidence status |
|---|---|---|---|---|---|
| Application shape (D) | Modular monolith: Next.js + FastAPI + PostgreSQL | Microservices; serverless functions; low-code tools | One small team, strong transactions for stock/approval, simple to deploy and debug | Many independent teams or very uneven scaling needs | Reasoning (architecture §4) |
| Database (D) | PostgreSQL, one database | MongoDB; SQLite; separate time-series DB | Transactions, constraints and row locks for exact inventory; mature; time-series extensions possible later | Single-user offline tool (SQLite) or huge telemetry volume (time-series DB) | Reasoning; EXP-14 |
| Research archive (D) | Compressed partitioned Parquet outside Git | Everything in PostgreSQL; CSV files | Fast columnar analysis, small size, reproducible manifests | Very small datasets | Reasoning |
| Numbers (D) | Exact decimals/integers for kg and ₹ | Floating point | Floats drift; stock and money must reconcile exactly | Never for business truth | Standard practice |
| Multi-FPO (D) | Tenant isolation from day one | Single-FPO app | Avoids a later rewrite; privacy | A one-off single-FPO prototype | Reasoning |

## Data and cleaning

| Area | Chosen | Alternatives | Why chosen | Alternative wins when | Evidence status |
|---|---|---|---|---|---|
| Market source (EL) | AGMARKNET via data.gov.in + CEDA history | Scraping websites; third-party aggregators | Official, permitted, long history | Official access denied, with permission for another source | EXP-01 |
| Cleaning (D) | Deterministic rules + flag-only statistical screen | ML anomaly detection; automatic deletion/clipping | Explainable; does not delete genuine price spikes | Large labelled error dataset exists | EXP-02 |
| Missing data (D) | Stays missing | Forward-fill; zero; interpolation as "observed" | Prevents fake confidence | Never for stored facts (models may impute internally, documented) | Reasoning |

## Forecasting and decisions

| Area | Chosen | Alternatives | Why chosen | Alternative wins when | Evidence status |
|---|---|---|---|---|---|
| Model selection (EL) | Evidence-based promotion among naive, ETS/SARIMA, XGBoost/LightGBM, Chronos-2, optional A3T-GCN | Pick one deep model up front | Prices are sticky; simple baselines often win; need fair comparison | — (selection is the experiment) | EXP-03 |
| Uncertainty (proposed, OI-02) | Required intervals for horizons > 7 days | Point forecasts only | Holding decisions need risk, not one number | Very short horizons with stable prices | EXP-04 |
| MSP crops (D, CHAT-20) | Fixed MSP price path, no model | Forecast MSP-crop mandi prices | Mandi prices sit at MSP; a model adds error, not value | Years when open-market prices clearly exceed MSP | Patiala research Part A |
| Hold/sell (proposed, OI-01) | Separate advisory comparing later price range vs storage risk and money cost | Fold into 7-day optimizer; no advice | Different time scale; keeps optimizer simple | — | EXP-07 |

## Storage monitoring

| Area | Chosen | Alternatives | Why chosen | Alternative wins when | Evidence status |
|---|---|---|---|---|---|
| Sensor (D, CHAT-06) | ESP32-WROOM-32 + protected SHT30 probe in the grain | DHT22; capacitive moisture cable; handheld moisture meter; CO₂ sensor | Low cost, digital I²C sensor with tighter datasheet RH accuracy than DHT22 (compare datasheets at purchase), protected probe, automatic readings; in-grain RH ≈ water activity | Budget allows commercial cables; CO₂ sensing for early spoilage detection is a strong future option | Sources in Patiala research B1–B2; EXP-13 |
| Risk rule (EL, CHAT-18) | Water-activity thresholds 0.65/0.70 with temperature adjustments and hysteresis | Moisture % via isotherm equations; ML classifier; ambient-only rule | Crop-independent, published thresholds, no paywalled constants, explainable | Isotherm constants verified (adds moisture display); labelled spoilage data exists (ML) | EXP-06 |
| Connectivity (D) | Wi-Fi via hotspot + MQTT over TLS | GSM module; LoRaWAN; manual upload | Cheap, works with existing phone coverage | No phone near the shed (GSM/LoRa) | EXP-13 |

## Optimisation and fairness

| Area | Chosen | Alternatives | Why chosen | Alternative wins when | Evidence status |
|---|---|---|---|---|---|
| Method (EL) | Exact MILP (SCIP first), benchmarked vs HiGHS, CP-SAT, greedy | Heuristics only; genetic algorithms; reinforcement learning | Integer quantities, linear costs, provable optimality and clear failure states | Problems too large for exact solving within time limits | EXP-08 |
| Objectives (D) | Lexicographic order: allocation → fairness → storage → net return → tie-break | Weighted sum (e.g. 0.6 profit + 0.25 storage + 0.15 fairness) | Weights mix rupees with ratios arbitrarily and hide trade-offs; order is explainable | Stakeholders agree on explicit exchange rates between goals | EXP-09 |
| Fairness (D) | Per-farmer leximin of fulfilment | Max-min only; proportional; utilitarian (max total); per-lot fairness | Protects the worst-off farmer first, then the next; a farmer with many lots gets no extra weight | FPO prefers rewarding larger members, or speed requires max-min only | EXP-09; PILOT-009 |
| Market load (EL) | Disabled/advisory in demo | Strict capacity limits | Arrivals are not official capacity | Validated mandi capacity data exists | CORE-008 |

## Manager workflow and communication

| Area | Chosen | Alternatives | Why chosen | Alternative wins when | Evidence status |
|---|---|---|---|---|---|
| Control (D) | Manager approves; approval reserves stock atomically | Fully automatic selling | Accountability; real-world variance; trust | — | Architecture §2 |
| Edits (D) | Immutable revisions + shared validator | Edit plan in place | Audit trail, reproducibility for the paper | — | Reasoning |
| Farmer channel (D, CHAT-16) | WhatsApp, recorded preferred language, reviewed templates | SMS; voice/IVR; app; live LLM translation | User requirement; widely used in India (actual coverage among member farmers unknown, OI-23); templates keep numbers exact | Farmer has no smartphone/WhatsApp (OI-23: SMS/IVR fallback) | EXP-12 |
| Sale evidence (EL) | Staff record from digital J-form (number = duplicate key) | Typed sale without receipt; sensor inference | J-form is official, QR-verifiable, already sent to farmers | Mandi Board API access becomes available (import directly) | Patiala research PILOT-005 |

## How to use this file

When a choice changes, add a row update with the date and decision ID rather than deleting the old reasoning. The paper's Methods section cites the "Why chosen" and Results cites the experiment.
