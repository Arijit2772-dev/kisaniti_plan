# Demo, test and deployment plan — a system you can open, use and trust

26 October 2026 • Requirement from the owner: everything tested, every integration checked, and at the end a website that shows all the working, running somewhere the owner can reach it. This plan extends [FINAL_DELIVERY_PLAN.md](FINAL_DELIVERY_PLAN.md) §3–§4. Classification: **EXISTS**, **TO BUILD**, **DECISION**, **UNKNOWN**.

## 1. Where the product is today (checked 26 Oct, not assumed)

`docker compose up --build -d --wait` brings up PostgreSQL, the API and the website. The website is **one page**: system status (database ready, migrations ready, DEMO banner). The API has three endpoints (`/health/live`, `/health/ready`, `/api/v1/meta`). PostgreSQL holds the tenant table and the sensor tables. CI is green on five jobs. **Every other screen (data, forecast, storage, plan, approval, WhatsApp) is TO BUILD.** Anything shown in the demo must come from a script or a real run; where a result does not exist yet, the page says "not yet computed" and shows nothing in its place.

## 2. What "show all the working" means: the demo path

One guided path through the website, on fictional FPO data (DEMO banner on every page, as today), each page showing where its data came from (real, policy, synthetic) and the version of the rule or model.

| # | Page | Shows | Source of what it shows | Depends on |
|---|---|---|---|---|
| D0 | System status | database, migrations, versions | API (EXISTS) | — |
| D1 | Market data | basmati months filed, row counts, reconciliation (raw = kept + flagged + removed), hand-check status, blank-date rule status | cleaning scripts' result files, imported by a job | B.1 (done), B.2 hand check (owner) |
| D2 | Prices and forecast | variety index chart, forecast with 80% and 95% ranges, "baseline is enough" when that is the result | forecasting results | C.0–C.7 (not started) |
| D3 | Storage | device list, latest reading, 24 h category, last seen, outage; live from the software node, later from the real node | ingestion + `core.storage` | H-4 to H-6 |
| D4 | Hold or sell | advice per lot with storage loss, holding cost, ranges, and its stated limits | hold-or-sell rule | D.1 (not started) |
| D5 | Plan | fair allocation across the synthetic members, price of fairness, validator result | allocation core | E (running), E.4 for basmati |
| D6 | Manager approval | review, edit, approve with atomic reservation, release | L4 | G.5 |
| D7 | Messages | the exact WhatsApp texts in three languages as previews; a real send only to the owner's own test number, only on explicit authorisation | L5 | G.6, test number |
| D8 | Sale and stock | enter a sale from a J-form, stock updates | L4/L5 | G.5–G.6 |
| D9 | Evidence | links to the paper's tables, the pre-registered protocol, test and CI status | repo files | — |

Honest consequence: until forecasting (C) and hold-or-sell (D) have results, D2 and D4 can show only what exists (the index and the baseline). The demo is built so that these pages light up when the results are produced, without redesign.

## 3. Test strategy: nothing counts as done without a recorded run

| Layer | What is tested | Today | To add |
|---|---|---|---|
| Algorithms (`core/`) | exact arithmetic, storage rule, allocation vs brute-force oracle, validator | EXISTS, CI | hold-or-sell rule with its seven branches (D.1) |
| Contracts | valid and invalid examples per schema | EXISTS, CI | slices for L1, L2, L3a, L3c, L4, L5 as they freeze |
| Device edge | firmware native tests; conformance harness; software node; live broker run | EXISTS (13 + 45 tests; live script) | Wokwi twin and the real node through `kn-conformance publisher` |
| API on real PostgreSQL | migrations, constraints, concurrency, idempotency | EXISTS (25 tests) | tenant isolation tests (two FPOs never see each other), reservation concurrency |
| Website | build, lint, types | EXISTS | **browser tests that click through D0–D9** (**DECISION**: a browser-test tool such as Playwright is a new dependency; the repo rule is no unnecessary dependencies, so the owner decides) |
| Whole stack | `docker compose up`, health, one scripted run of the demo path | health only | a script that starts the stack, feeds the software node, runs the demo path, and fails on any broken step (I.1) |
| Research numbers | every paper number from a script | partly | `paper/numbers.py` check before submission (G4) |
| Fresh machine | the demo starts from a fresh clone | not done | I.3 |

An integration is "checked" when a script crosses the boundary for real (a real broker, a real database, a real browser) and the result is stored with its commit. The conformance harness and the live broker run are the model for this.

## 4. Where it runs, and how you reach it

| Option | What it needs | Reach | State |
|---|---|---|---|
| **A. Local stack on this Mac, shown to you in the app's browser pane** | nothing new (Docker is installed; the stack is running now) | you, through this session | **available now** (opened at http://localhost:3000) |
| **B. Temporary public link from this Mac** (a tunnel tool) | downloading and running a tunnel program; the link works only while this Mac and the stack run; it exposes the DEMO (fictional data) to anyone with the link | anyone with the link | **DECISION** (a download and an outward-facing action) |
| **C. A cloud host or college server** | an account you create and own (and any payment); I cannot create accounts or enter payment details; the same `compose.yaml` runs on a plain Linux server | anyone, permanently | **DECISION**, needed before submission anyway for the paper's availability statement |
| **D. Static snapshot as a shareable page** | none | read-only; no live backend | possible for a record, not a substitute for a running system |

Rules for any public deployment: DEMO mode with fictional data only; no real farmer data; secrets in environment variables, never in the repo; HTTPS; the WhatsApp token never on a public demo; the privacy contract and a legal review come before any real pilot (G.8).

## 5. Steps, in order

1. **Now (no decisions needed):** keep the local stack as the working demo base; add the D1 page (market data and reconciliation) since its data exists; start H-4.
2. **Device path:** H-4 registry and tenant isolation, H-5 scheduled assessment, H-6 live view, so D3 runs from the software node with no hardware.
3. **Results pages:** D5 from the allocation experiments (after the solver-time decision); D2 and D4 as C and D produce results.
4. **Approval and messages:** D6, D7, D8 (needs the WhatsApp test number for a real send).
5. **Whole-stack script and fresh-clone run** (I.1, I.3); fix whatever breaks.
6. **Deploy** to the chosen target; run the same script against it; record the URL, commit and date in the paper's availability statement.
7. **Real node:** when the kit is built, the same pages show it with no code change (the plug-in rule).

## 6. Decisions needed from the owner

1. Where to deploy (§4, B or C), and who owns the account.
2. Whether to add a browser-test dependency (§3).
3. The earlier open decisions in [OPEN_ITEMS.md](OPEN_ITEMS.md) (protocol freeze, sub-yard rule, demo MSP season, venue and deadline).
