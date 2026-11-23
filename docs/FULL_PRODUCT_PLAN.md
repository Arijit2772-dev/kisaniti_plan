# Full product plan: five phases to a public URL (23 Nov 2026)

> **Decision CHAT-53 (23 Nov):** real WhatsApp (Meta Cloud API) and a public domain are required. The owner buys the domain, hosting and Meta setup. Real sends go only to friends' numbers who agree, attached to fictional farmers; numbers never go into git or chat.

**Progress:** 1.1 done (`9a35f07` DHT11 firmware, `cefeb03` HW-S2). Next: 1.3 (HW-S3).

Goal: anyone with the URL can use KisanNiti on a phone or a laptop; the owner's DHT11 node (CHAT-52) sends real readings into the hosted system; every workflow (sign-in → members/lots → sensor → storage category → plan → approval → handover → sale → report) runs, is recorded in the database, and is tested. The owner supplies (1) the hardware build, guided by the agent, and (2) one design sheet per page. Everything else is the agent's work, item by item, under `BUILD_LEDGER.md` rules (contract first, tests, report, ledger tick, commit only own files).

Decisions used: up to CHAT-53. Items marked **NEEDS OWNER** stop until the owner answers.

## Phase 1 — software that needs no hardware (agent; starts now)

| # | Item | Done when |
|---|---|---|
| 1.1 | Review the uncommitted HW-S2 files (`scripts/provision_node.py`, `backend/tests/test_provision_node.py`, `scripts/hw_stack_check.sh` edit), then commit the DHT11 firmware (HW-S0) on its own | 3 envs build, native tests pass, CI green |
| 1.2 ✅ | HW-S2 provisioning: broker user + generated git-ignored `secrets.h`; Wi-Fi password typed by the owner only | build with a generated file succeeds; certificate check passes |
| 1.3 | HW-S3 `kn-ingest` survives a PostgreSQL restart | test restarts the DB, zero readings lost |
| 1.4 | HW-S4 `scripts/hw_preflight.sh` | PASS/FAIL per line for stack, broker, worker, cert host vs IP, device, newest reading |
| 1.5 | Logging and build identity: JSON logs, request ID end to end, commit hash in `/api/v1/meta` and the footer | a request ID is traceable from browser to DB log line |
| 1.6 | Browser tests (K.8): Playwright on a phone size (375 px) and a laptop size, through every workflow; **NEEDS OWNER: yes/no to adding Playwright as a dev dependency** | suite green locally and in CI |
| 1.7 | HW-S5 full rehearsal from a fresh clone with the software node | recorded run, every step PASS |

**Gate 1:** `chain.py`, smoke, browser tests and HW-S5 all pass on a fresh clone.

## Phase 2 — the real node (owner builds, agent guides and connects)

DHT11 wiring (CHAT-52): VCC → 3V3, DATA → GPIO 4, GND → GND, no extra resistor (the module has one). Owner sends a photo before power-on.

| # | Item | Who |
|---|---|---|
| 2.1 | HW-A photo check of board, module, wires | owner photo → agent confirms |
| 2.2 | HW-B bench check on the breadboard | owner, agent guides |
| 2.3 | HW-C provision + flash `esp32dev_dht11`, reading visible on the local website | agent, owner plugs USB and types Wi-Fi password |
| 2.4 | HW-D `kn-conformance` against the real board, no FAIL; close BUG-009 on real hardware | agent |
| 2.5 | HW-E outage tests (Wi-Fi off, broker off, DB off → buffered readings arrive, none duplicated) with records in `firmware/records/` | both |
| 2.6 | Link the node to a lot; the lot shows a storage category; plan uses it | agent |

**Gate 2:** real records saved; readings labelled DHT11 (demo only, not paper results — O75).

## Phase 3 — remaining features that need owner approvals

**NEEDS OWNER** before any code:

- **O70 / model comparison (Slice 6):** confirm or change the 8 evaluation choices in `rp:layers/L3a_forecasting/EVALKIT_ASSUMPTIONS.md` (DM loss = absolute error; origin day counted; seasonal naive on the target's day ±3; moving average same season only; Bartlett weights; bootstrap blocks of h pairs; MASE scale from all earlier seasons; no range when too few residuals), and pick how §4 is corrected (1509 has 57 and 51 pairs at 7 and 14 days, not ≥ 60).
- **P-1 hold-or-sell** (`design/HOLD_SELL_ADVISORY.md`): values or evidence for loss rate per storage category ℓ, holding cost k, money rate r, margin τ, range-width limit w_max; how advice beyond 28 days is handled (O3; evidence stops at 28 days); the promotion rule (which model counts as PROMOTED) after the comparison runs.
- Smaller open choices that change behaviour if touched: O51, O52, O54, O55, O61. Default: leave as today and label DEMO.

Then: freeze contract → build → tests → report, one item at a time.

**Gate 3:** chain extended to hold-or-sell; all suites green.

## Phase 4 — deploy to a public URL

**NEEDS OWNER** (accounts and payments; the agent never creates accounts, types passwords or enters card details):

1. **Server**: one Linux VM, Ubuntu 24.04, at least 2 vCPU, 4 GB RAM, 40 GB disk, public IPv4, ports 22/80/443/8883 open. Any provider you pay for (Azure for Students, AWS Lightsail, DigitalOcean, Hetzner…). Check the price on the provider's page. Give me SSH access with a key (I give you the public key; no password in chat).
2. **Domain**: any registrar. Point two DNS A records at the server IP: `app.<domain>` (website) and `mqtt.<domain>` (sensor broker). A real domain gives a Let's Encrypt certificate, which the ESP32 can check; that fixes the IP-only certificate problem (O36).
3. **WhatsApp (required)**: Meta Business account → Meta developer app with the WhatsApp product. Start with Meta's free test number and add each friend's number as a test recipient in the dashboard (they confirm a code there; the limit is shown in the dashboard). For later: your own business number (a new SIM is safest; a number in the normal WhatsApp app may not be usable with the Cloud API — check in the dashboard) and a payment method in Meta. You put the access token and the app secret on the server yourself, or into a prompt I give you; never in chat.
4. **Templates**: you submit the three message templates (English, Punjabi, Hindi; texts already reviewed) for approval in the dashboard; I give you the exact text and variables.

Agent then, each as its own item with tests and a report:
- 4.1 Production compose: Caddy (automatic HTTPS) in front of website and API; PostgreSQL not exposed; secrets in a server-only `.env`; nightly database backup with a restore test; uptime check; deploy script that deploys one git commit and can roll back to the previous one.
- 4.2 Broker on `mqtt.<domain>:8883` with the public certificate; re-provision the node (`provision_node.py`) for the public host.
- 4.3 Public demo FPO with fictional data, demo login on the sign-in page, nightly reset; visitors cannot send WhatsApp.
- 4.4 WhatsApp channel (S4.5) behind the existing `messaging/v1` contract: sends only approved, reviewed templates to numbers with consent; status webhook (S4.6) at `https://app.<domain>/…` with signature check; delivery records update. Contract amendment first (it is a new channel), then code.
- 4.5 Run `chain.py`, smoke and browser tests against the public URL from a phone and a laptop; one real send to a friend's number, delivered status seen on the website.

**Gate 4:** every test passes against the public URL; restore from backup proven once.

## Phase 5 — UI from the owner's designs, then the demo

The owner sends one design sheet per page; the agent builds it without changing behaviour, then re-runs all tests on phone and laptop sizes. What each page must contain is listed in §6 below.

Then P-4: demo script (guide §6), run twice on different days with the node in grain ≥ 30 h before, a recorded fallback (software node + screen recording) if the real node or Wi-Fi fails during the video.

**Gate 5:** two clean rehearsals recorded.

## 6. What each page must show (for the design sheets)

Every page: language switch (Punjabi/English), signed-in user and FPO, sign-out, build identity in the footer, loading/empty/error states, works at 375 px.

| Page | Sections and elements that must exist |
|---|---|
| Sign-in | user name, passphrase, sign-in button, error message, language choice; (if approved) demo login hint |
| Change password | current, new, repeat, rules, result |
| Home | FPO name, counts (members, lots, stock), newest sensor state, plans waiting for review, links to each area |
| Users | list (name, role, active), add user, change role, deactivate |
| Status | API, database, broker/worker, newest reading age, build/commit |
| Members | list with search, add/edit member, active flag, member's lots |
| Lots | list (member, crop, variety, quantity kg, place, storage category, reserved/available), add lot, stock ledger entries per lot, link a sensor |
| Prices | mandi and variety filters, price table with dates, dataset status, MSP, newest date and its age |
| Storage sensors | devices, assignment to a place, latest temperature and humidity, chart over time, storage category with reason, sensor model label (DHT11 = demo) |
| Plans | price outlook with ranges, create plan, proposal with explanations, edit/revision, review box (approve/reject with reason), reservation, release, instructions, execution and report |
| Hold-or-sell (Phase 3) | per lot: recommendation, gain range, storage reason, "information only" note |
| Contacts | masked phones, consent state and events, access log |
| Messages | templates in three languages, preview, handover one-tap links, sent/not sent marks, delivery records |
| Sales | record sale against a reservation, off-plan sale, void, planned vs actual |

## 7. What the owner gives, in one list

1. Yes/no: Playwright (1.6).
2. Photos and hands for Phase 2.
3. O70 answers and P-1 parameter values (Phase 3).
4. Server, domain, Meta account, template submission (Phase 4, §Phase 4 list). The "who may use it" decision is taken: public demo FPO with fictional data (CHAT-53).
5. One design sheet per page (Phase 5).
6. Return the DHT11 order and reorder the SHT30 if paper measurements are still wanted (O75).
