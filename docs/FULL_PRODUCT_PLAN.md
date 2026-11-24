# Full product plan, by architecture layer (revision 2, 23 Nov 2026)

> **Decisions used:** up to CHAT-53 (real WhatsApp and a public domain required; real sends only to friends' numbers who agree, on fictional farmers). Hardware is the DHT11 stop-gap (CHAT-52) "as it is": demo readings, not paper measurements.

**Goal.** Anyone with the URL can use KisanNiti on a phone or laptop. Data flows through all five layers and back: L1 data from four sources → L2 cleaning → L3 forecasts, storage risk, hold-or-sell and the fair optimiser → L4 manager review and edits → L5 multilingual WhatsApp to farmers → actual sales recorded and compared with the plan. Every step is stored, tested, committed and reversible through git history.

**Who does what.** Agent: everything below except what is marked **OWNER**. Owner: buys the few items in §4, builds the node with the agent's guidance, gives one design sheet per page, answers the approvals in §3.

**Progress (24 Nov):** HW-S0 `9a35f07`, HW-S2 `cefeb03`, HW-S3 `acf6372`, HW-S4 `a37592f`. Phase 1: logging and build identity `edaf378`, forecast models page `d21584a`, browser tests at 375/1280 px `8431196`. Phase 3: server setup `6b0333d`, production stack and deploy/rollback `de54129`, nightly backup with proven restore `fc7976e` (leak fix `af7f01b`), trial on the real server: smoke PASS, about 500 MB RAM free. Waiting on: domain DNS (owner, YesDomain ticket), approvals A1-A3, Meta WhatsApp setup, page designs, the DHT11 node.

## 1. Where each layer stands (checked 23 Nov in the repos, not assumed)

| Layer | What the architecture asks | Built and tested | Missing for the full product |
|---|---|---|---|
| **L1 Data, 4 sources** | (1) AGMARKNET mandi prices, (2) official MSP table, (3) storage sensor readings, (4) the FPO's own records (members, lots, stock, sales) | All four enter the product: price release import with refusals and VALIDATED status (market/v1); MSP table from PIB (18 values); sensor MQTT → `kn-ingest` → PostgreSQL (L3B v1, isolation tests); members, lots, exact stock ledger (org/v1) | A way to add **new** price data on the hosted site (AGMARKNET needs a CAPTCHA, so scripts cannot download): see approval A2 |
| **L2 Cleaning and preprocessing** | Deterministic parsing, flags not deletion, hand check, daily variety index | Research repo: parser B.1 (6,678 rows reconcile with 29 manifests), hand check 56/56 signed off, index B.4 | The same parser and checks run **inside the product** when a new file is uploaded (today the cleaned release is made in the research repo and imported) |
| **L3a Forecasting** | Pre-registered model comparison; a model is used only if it beats the baseline | Protocol frozen 27 Oct with your approval; ETS, ARIMA, LightGBM, GRU, Chronos-2 vs baselines; test season 2025-26 run once. **Result: 42 of 42 comparisons NO_DETECTABLE_DIFFERENCE** against the naive forecast. Product uses the naive forecast with ranges | Product shows each model's status (NOT_PROMOTED, with the reason and the evidence file) instead of only "naive"; the plan keeps the naive forecast, as the evidence says |
| **L3b Storage risk (IoT)** | 24 h median + hysteresis → LOW/MEDIUM/HIGH/UNKNOWN per lot | Assessment job, placement at reading time, endpoints, storage page; DB-restart safe (HW-S3) | Real node connected (Phase 2); DHT11 label on the page |
| **L3c Hold-or-sell** | Per-lot advice for held basmati/wheat, information only | Design approved in principle (`design/HOLD_SELL_ADVISORY.md`); not built | Contract + build after approval A1 |
| **L3d Optimiser + fairness** | Exact allocation to mandis under capacity, max-min fairness, validator, explanations | plan/v1: optimiser, independent validator, immutable proposals, coded explanations | Paper experiments unfinished (solver time limit, O29) — research only, not needed for the product |
| **L4 Manager review** | Review, edit (revision), approve with exact reservation, reject, release | approval/v1 with concurrency and isolation tests | — |
| **L5 Farmers, multilingual** | WhatsApp in Punjabi, Hindi, English; consent; delivery records | Templates in 3 languages REVIEWED (28 Oct); contacts with consent and access log; preview channel; manager one-tap `wa.me` handover (CHAT-49) | **Meta Cloud API send, status webhook, farmers' replies** (S4.5, S4.6, O66), Phase 4 |
| **Actuals and outcomes** | Real sales consume the reservation; planned vs actual | sales/v1: sales, off-plan reconciliation, voids, report | Replies arriving by webhook instead of typed in (Phase 4) |
| **Across layers** | Auditability, explicit failures, isolation | One-command chain `chain.py` (13 steps, sabotage run), 365 backend tests, CI 5 jobs | Structured logs + request ID + build identity; browser tests on phone/laptop; hosting, backups, monitoring; owner's UI |

## 2. Five phases

Each item: contract first where behaviour changes, tests, a report in `docs/reports/`, one commit, CI green, ledger tick. Rollback is `git revert` of that commit; deploys name one exact commit (Phase 3).

### Phase 1 — finish the software end to end (agent, now; no purchases)
1. HW-S4 `scripts/hw_preflight.sh`: PASS/FAIL per line for stack, broker, worker, certificate host vs IP, device, newest reading.
2. Logging and build identity: JSON logs, one request ID from browser to database log line, commit hash in `/api/v1/meta` and the footer.
3. L2 in the product: upload a new AGMARKNET export on the prices page → the research parser's rules run server-side (same tests, same refusals) → new rows appear as PENDING until a person marks them checked. Needs A2.
4. L3a model status in the product: read-only model registry from the frozen results (all NOT_PROMOTED, with the evidence file and date).
5. L3c hold-or-sell (contract `holdsell/v1`, then code, page section on lots and plans). Needs A1.
6. Browser tests (Playwright) at 375 px and laptop width through every workflow, in CI.
7. HW-S5 full rehearsal from a fresh clone with the software node. **Gate 1:** chain, smoke, browser tests, HW-S5 all pass.

### Phase 2 — the real DHT11 node (owner builds, agent guides and connects)
Wiring: DHT11 VCC → 3V3, DATA → GPIO 4, GND → GND (CHAT-52). Photo check → bench check → `provision_node.py` + flash `esp32dev_dht11` → `kn-conformance` on the real board → outage tests (Wi-Fi, broker, database) with records in `firmware/records/` → link to a lot → storage category appears in the plan. **Gate 2:** real records saved, labelled DHT11.

### Phase 3 — hosting on the public URL (needs the server and domain from §4)
Images built in GitHub Actions and pushed to GitHub's container registry (free for public repos), so the server only pulls them: a small server is enough and nothing is built on it. Caddy for automatic HTTPS on `app.<domain>`; broker on `mqtt.<domain>:8883` with the same public certificate (the ESP32 checks it); PostgreSQL not exposed; secrets only in a server `.env`; nightly backup with one restore proven; uptime check; deploy and rollback by commit hash; public demo FPO with fictional data, demo login on the sign-in page, nightly reset (CHAT-53). Node re-provisioned to the public broker. **Gate 3:** chain, smoke and browser tests pass against the public URL from a phone and a laptop.

### Phase 4 — real multilingual WhatsApp (needs the Meta app from §4)
Contract amendment for the Cloud API channel inside `messaging/v1` (only REVIEWED templates, only consented numbers, explicit failure states) → send through the Meta test number → status webhook at `https://app.<domain>/…` with signature check → delivery records update → farmers' replies arrive and are recorded (O66). Your friends' numbers are typed only into the hosted site and the Meta dashboard. **Gate 4:** one plan goes all the way: approve → WhatsApp in Punjabi, Hindi and English to friends' phones → delivered status on the site → reply recorded → sale → planned vs actual.

### Phase 5 — your UI, then the demo and video
You send one design sheet per page (list in §5); I build each without changing behaviour and rerun every test at both sizes. Then the demo script, rehearsed twice on different days, node in grain at least 30 h before; fallback ready (software node + screen recording). **Gate 5:** two clean rehearsals recorded.

## 3. Approvals I need (one word each is enough)

- **A1 Hold-or-sell v1 follows the evidence.** No forecasting model was promoted (42/42 no detectable difference), so by the approved design's own rules the advice is: SELL_NOW (storage) when the lot's storage is HIGH; NO_RELIABLE_SIGNAL otherwise; UNAVAILABLE when there is no usable price. The loss rate, holding cost, interest rate, margin and range limit are only needed once a model is promoted, so they stay **unset** (not invented). Approve: yes / no.
- **A2 How new price data enters the hosted site.** (a) You download the monthly AGMARKNET export in your browser (you solve the CAPTCHA) and upload it on the prices page; or (b) you register a free data.gov.in API key and I test whether its feed covers Punjab basmati by mandi and variety, falling back to (a). Recommended: (a) now, (b) later.
- **A3 Everything else stays as it is and labelled DEMO** (plan policies O48, approver role O54, freshness warnings O55, reconciliation O63, inactive members O52, consent and planning O61). Approve: yes / no.

## 4. What to buy (cheap or free)

| Item | Cheapest that works | Cost |
|---|---|---|
| Server | **Azure for Students** (https://azure.microsoft.com/en-in/free/students): USD 100 credit for a year, no card; check the VM price in the portal. If not eligible: DigitalOcean Basic, Bangalore region (https://www.digitalocean.com/pricing/droplets), 2 GiB for the demo or 4 GiB to be safe | free / USD 12 or 24 a month (DigitalOcean page, 23 Nov) |
| Domain | Any registrar, e.g. https://www.namecheap.com/ or https://www.cloudflare.com/products/registrar/ ; a `.in` or `.com` name | about a year's fee; check the registrar |
| WhatsApp | Meta Business (https://business.facebook.com/) + developer app with WhatsApp (https://developers.facebook.com/apps/). The test number needs no payment method for template messages (Meta docs). Recipient limit for the test number: check the API Setup page | free for testing |
| Hardware | nothing more; DHT11 + ESP32 + breadboard you have | 0 |

## 5. What each page must show (for your design sheets; full brief: `docs/UI_DESIGN_BRIEF.md`)

Every page: Punjabi/English switch, signed-in user and FPO, sign-out, build version in the footer, loading/empty/error states, works at 375 px.

| Page | Must contain |
|---|---|
| Sign-in | user name, passphrase, error message, language; demo login hint |
| Change password | current, new, repeat, rules, result |
| Home | FPO name, counts, newest sensor state, plans waiting for review, links |
| Users | list (name, role, active), add, change role, deactivate |
| Status | API, database, broker and worker health, newest reading age, build version |
| Members | searchable list, add or edit, active flag, the member's lots |
| Lots | member, crop, variety, kg, place, storage category, reserved and available, stock ledger, sensor link, hold-or-sell advice |
| Prices | mandi and variety filters, price table, dataset status, MSP, newest date and age, **upload new export** (A2) |
| Forecast models | each model's status (NOT_PROMOTED and why), evidence date |
| Storage sensors | devices, placement, latest temperature and humidity, chart over time, category with reason, "DHT11 = demo" label |
| Plans | price outlook with ranges, proposal with explanations, edit, approve or reject with reason, reservation and release, instructions, execution, report |
| Contacts | masked phones, consent history, access log |
| Messages | templates in three languages, preview, WhatsApp send and delivery status, one-tap fallback links, replies |
| Sales | sale against a reservation, off-plan sale, void, planned vs actual |
