# Open issues that can cause later problems

15 October 2026 • Review of the whole folder after the Patiala research and the demo crop decision (CHAT-20). Each issue says **why it bites later**, what we recommend, and **who must decide or act**. Severity: **S1** = blocks correct product or real-world use; **S2** = causes rework if ignored; **S3** = quality/maintainability.

Nothing here closes an `UNKNOWN`. Agents: when you resolve an issue, record the decision in the [decision register](blueprint/01_DECISION_REGISTER.md) and mark it here as resolved with a link. Do not delete rows.

## A. Design gaps exposed by the crop decision

| ID | Sev | Issue | Why it causes problems later | Recommendation | Decides / acts |
|---|---|---|---|---|---|
| OI-01 | S1 | **Hold-or-sell decision is not modelled.** The optimizer plans mandi/date/quantity inside a ~7-date horizon (CHAT-11), but held wheat's real question is *"sell now or keep it until July/October?"* | Short planning windows cannot express a months-long holding choice; the demo's main value would be missing or faked. | Add a separate **Layer 3 hold/sell advisory**: compare expected net price later (forecast with uncertainty) minus storage-risk loss and money cost of waiting versus selling now. Then run the existing optimizer for the chosen near-term sale window. Specify it as a new contract, compare policies in [EXP-07](research/RESEARCH_AND_EXPERIMENT_PLAN.md). | User approves the idea; agent specifies it |
| OI-02 | S1 | **Forecast horizon and uncertainty.** Held wheat and basmati need 30–120-day forecasts; the current contract has a point forecast with optional intervals. | Long-horizon point forecasts are unreliable. Advising a farmer to hold based on one number can lose money. | Make prediction intervals/quantiles **required** for horizons > 7 days. Evaluate interval coverage. Allow the advisory to say "no reliable signal — sell when convenient". | Agent (contract) |
| OI-03 | S1 | **Basmati data quality.** AGMARKNET basmati series in Patiala mandis are sparse (e.g. one day of 10 t at Patran) and variety names are inconsistent. | Models trained on sparse, mislabelled series give confident nonsense. | Measure coverage first ([EXP-01](research/RESEARCH_AND_EXPERIMENT_PLAN.md)). If sparse, forecast at Punjab/Haryana regional level plus a per-mandi difference. Build a reviewed variety-name mapping. | Agent, after data access |
| OI-04 | S2 | **MSP fixed-price path details** (grade A vs common, value cuts, relaxations, procurement windows, held wheat sold to agencies outside the window?). | Wrong windows give impossible plans; ignoring value cuts overstates income. | Define the `MSP_SCHEDULE` contract with season, grade, window and source. Treat value cuts as unknown unless announced. | Agent; confirm with arhtiya |

## B. Data access and legal

| ID | Sev | Issue | Why it causes problems later | Recommendation | Decides / acts |
|---|---|---|---|---|---|
| OI-05 | S1 | **No working market-data access yet.** Earlier probes hit login/CAPTCHA. The data.gov.in "current daily price" resource page says its API does not exist (request needed) ([data.gov.in](https://data.gov.in/resource/current-daily-price-various-commodities-various-markets-mandi)); a variety-wise resource also exists ([data.gov.in](https://www.data.gov.in/resource/variety-wise-daily-market-prices-data-commodity)). CEDA (Ashoka) offers 2000-onward history with monthly refresh ([CEDA](https://agmarknet.ceda.ashoka.edu.in/README.html)). | Without data there are no models, no experiments, no paper results. The 06:00/18:00 schedule (CHAT-04) may be impossible if the source only refreshes monthly or needs approval. | **User registers** on data.gov.in (and requests API access for the variety-wise resource) and on CEDA. Then an agent runs EXP-01 to measure what is really available. Never scrape around CAPTCHAs. | **User** (account creation is a personal action) |
| OI-06 | S1 | **Indian data-protection law is not in the design.** The DPDP Act 2023 and DPDP Rules 2025 (notified 13–14 Nov 2025; most obligations about 18 months later, ~May 2027) cover consent notices, purpose limits, security safeguards and breach reporting ([PIB](https://static.pib.gov.in/WriteReadData/specificdocs/documents/2025/nov/doc20251117695301.pdf); [Deccan Herald](https://www.deccanherald.com/amp/story/india%2Fcentre-notifies-dpdp-rules-implementation-planned-in-phases-spread-over-12-18-months-3798465)). KisanNiti stores farmer names, phones, locations, sales and WhatsApp consent. | Retro-fitting consent, deletion and security later is expensive; a real pilot could be unlawful. | Add a privacy contract before coding: data inventory, plain-language consent notice in Punjabi/Hindi/English, purpose limits, retention/deletion, access logging, breach procedure. **Never store Aadhaar numbers**; store only what planning needs. Get legal review before any real pilot. | Agent drafts; user/legal adviser approves |
| OI-07 | S2 | **WhatsApp Business setup and cost.** A sender needs a verified business (the FPO) and approved templates; business-initiated messages are charged. | Without an FPO entity and budget, the "required" live channel cannot pass WA12. | Use a Meta test number for development; plan who owns the real sender. Record message-cost estimates once the account exists. | User / FPO |
| OI-08 | S3 | **Data licence and attribution.** data.gov.in and CEDA have terms of use. | A paper or app can be challenged if terms are breached. | Record terms in each dataset manifest; cite sources in the paper. | Agent |

## C. Engineering foundations

| ID | Sev | Issue | Why it causes problems later | Recommendation | Decides / acts |
|---|---|---|---|---|---|
| OI-09 | S1 | **RESOLVED 15 Oct 2026:** local Git repositories (branch `main`) initialised in both folders with a first commit; no remote yet (private GitHub remote still recommended). Original issue: **Neither the planning folder nor `kisaniti_implementation` is under version control** (no Git). | Lost work, no history, no way to review agent changes or reproduce paper results. | `git init`, first commit of the current docs, private GitHub remote; large data outside Git (already the rule). | User approves; agent runs |
| OI-10 | S1 | **Coding gate still closed** ([blueprint status](blueprint/00_BLUEPRINT_STATUS.md): product coding PROHIBITED). | Agents must refuse code; experiments needing code stall. | User explicitly authorises Phase 1 once W01 contracts are frozen. Experiments EXP-01/02 (data measurement) can run as research scripts after authorisation. | **User** |
| OI-11 | S2 | **No pinned technology versions** (Python, FastAPI, Next.js, PostgreSQL, SCIP/PySCIPOpt, HiGHS, OR-Tools, XGBoost, Chronos-2, ESP32 toolchain). | "Works on my machine", irreproducible experiments. | Pin versions with lockfiles in W02 and record them in every experiment report. | Agent |
| OI-12 | S2 | **Login, hosting, MQTT broker, backup and running cost not chosen.** | The ESP32 needs a reachable TLS broker; FPO staff need a login; the demo needs somewhere to run. | Choose a simple setup: one small cloud VM or a laptop + tunnel for the demo, managed PostgreSQL backups, phone-OTP or password login. Write down cost per month. | Agent proposes; user approves cost |
| OI-13 | S2 | **Manager app language and device.** Design assumes a web app; FPO staff mostly use phones and may prefer Punjabi. | Low adoption, failed usability test. | Mobile-first web UI; Punjabi/Hindi/English UI labels (separate from farmer message language). | User confirms; agent designs |
| OI-14 | S2 | **Fairness computation size.** Exact leximin adds about F² helper variables (1,000 farmers ≈ 1,000,000). | Solver may be too slow for large plans. | Benchmark in EXP-08; test the faster equivalent "progressive bottleneck" method; report realistic per-plan sizes. | Agent |
| OI-15 | S3 | **Documentation volume and overlap** (~520 KB, many superseded drafts). | Agents may read an old draft and implement the wrong thing. | After contract freeze, write one consolidated `SPEC.md` and mark older docs as history. Until then, follow [START_HERE](START_HERE.md) precedence. | Agent |

## D. Real-world validity

| ID | Sev | Issue | Why it causes problems later | Recommendation | Decides / acts |
|---|---|---|---|---|---|
| OI-16 | S1 | **No contact with a real FPO yet.** All 21 unknowns are desk-research guesses. | The product can be technically perfect and still unused. | One 30-minute talk with a Patiala FPO CEO and one arhtiya, using the questions in architecture §66 and [Part C](research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md). | **User** |
| OI-17 | S1 | **Value is unproven.** For MSP crops the price is fixed; for basmati/held wheat, gain over "sell at harvest at the nearest mandi" is unknown. | A paper claiming benefit without a backtest is not credible; real users lose trust. | Run the historical backtest [EXP-10](research/RESEARCH_AND_EXPERIMENT_PLAN.md) against simple baselines and **publish whatever it shows**, including "no gain". | Agent, after data |
| OI-18 | S2 | **Farmer–arhtiya credit ties.** Farmers often sell through their own commission agent. | Plans may be ignored; off-plan sales common (CORE-009). | Design the advice to work *with* the farmer's arhtiya (choose date/mandi, not the agent). Measure adherence in the pilot. | User/FPO |
| OI-19 | S2 | **Storage-risk policy unvalidated.** | Wrong HIGH/LOW labels push early or late sales. | Salt calibration, bench settling test, one-time mandi moisture comparison ([B5](research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md)). | User (hardware) + agent |
| OI-20 | S2 | **Research ethics.** Interviews or surveys with farmers/FPO staff for a paper may need college ethics approval and informed consent. | Unusable results or a rejected paper. | Ask the college guide; prepare a consent form before any interview. | User |
| OI-21 | S2 | **Research contribution not yet defined; no literature review.** | A paper needs a clear question and novelty. | Proposed contributions are in the [paper outline](paper/KisanNiti_Paper_Outline.md); a literature review is task EXP-00. | User + agent |
| OI-22 | S3 | **One sensor represents one bag only.** | Overstating coverage misleads the manager. | Show assessment scope per lot; unmonitored lots stay UNKNOWN (already designed). | — (keep rule) |

## What "ready for real-world use" means here

Documents cannot make the system ready for real-world use. It is ready only when **all** of these have recorded evidence:

1. The code exists and passes its acceptance tests on real PostgreSQL (T01–T48, L4T, L5T, WA).
2. Real market data has been imported and models evaluated (EXP-01…06).
3. The optimizer has been checked against a brute-force reference and benchmarked (EXP-08/09).
4. The backtest shows when the advice helps and when it does not (EXP-10).
5. The hardware bench test and the storage-policy checks have passed.
6. WhatsApp live tests have passed with consenting test recipients.
7. The privacy contract has been implemented and legally reviewed.
8. A small pilot with one FPO has been run with its outcomes recorded.

Until then the honest label is **"working demonstration"**, not "production ready".
