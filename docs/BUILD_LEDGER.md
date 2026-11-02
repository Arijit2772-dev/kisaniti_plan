# Build ledger — where the work stands, so nothing is done twice

Read this first in a new session, after `NEW_CHAT_START.md`. Updated at the end of every working step. Repos are in `~/Documents` (`kisaniti`, `kisaniti_implementation`, `kisaniti_research_paper`); the session folder `sem_7/kisaniti` is an old copy and must not be used. The owner will change the UI later: do not polish pages; build function and tests.

## 1. Done (with evidence)

| Area | State | Where | Evidence |
|---|---|---|---|
| Shared contract, FPO table | frozen, built | product `contracts/shared/v1`, migration 0001 | CI |
| L3b sensor path: contract, firmware, `core.storage`, MQTT ingestion | built, tested | product `firmware/`, `backend/ingestion/`, `core/` | 45 ingestion tests; live broker script |
| H-1 conformance harness, H-2 software node, H-3 PostgreSQL readings | done | `backend/ingestion/src/kisaniti_ingestion/{simulator,conformance}.py`, migration 0002 | report `docs/reports/2026-10-26_H1-H3_*` |
| Access slice (login, sessions, roles, two walls) | done, contract `access/v1` frozen + amendment A1 | `backend/src/kisaniti_api/access/`, migration 0003 | 82 backend tests; report `…_access_login_two_fpo.md` |
| H-5 storage assessment job + endpoints | done | `backend/src/kisaniti_api/devices/{assess_job,placement}.py`, migration 0005, compose `assessor` | 121 backend tests; `scripts/demo_storage.sh` |
| H-4a devices/v1: registry, assignments, tenant-scoped readings, DB registry for kn-ingest | done | `backend/src/kisaniti_api/devices/`, migration 0004 | 106 backend tests; live run with `REGISTRY=db` |
| Website: sign-in, change password, home, users, status; Punjabi and English; API proxy | done (UI to be restyled by the owner) | `frontend/src/` | report `…_signin_shell_punjabi.md` |
| Slice 1: members, places, lots, exact stock ledger (org/v1); mandis, MSP and price import with refusals (market/v1); pages: members, lots, prices, storage sensors (H-6) in Punjabi and English | done; dataset is PENDING_HAND_CHECK | `backend/src/kisaniti_api/{org,market}/`, migrations 0006 and 0007, `frontend/src/components/{Members,Lots,Price,Devices}*`, `scripts/make_ci_release.py` | 175 backend tests, 240 contract checks; report `…_slice1_foundation_data.md`; BUG-007 fixed |
| Slice 2: baseline forecast with ranges (core), sensor link, immutable plan proposals through the optimiser and validator, coded explanations (plan/v1); pages: plans, price outlook, sensor link on lots | done; DEMO policies, forecast not validated | `core/src/kisaniti_core/forecast.py`, `backend/src/kisaniti_api/plan/`, migration 0008, `frontend/src/components/{PlansPanel,LotsPanel}.tsx`, `frontend/src/app/[lang]/plans/` | 209 backend tests, 533 core tests, 277 contract checks, smoke through the website; report `…_slice2_first_plan.md` |
| Slice 3: plan revisions (validator-checked edits), approval with atomic exact reservation, rejection, release (approval/v1); review box on the plan page | done; demo roles and reason codes | `backend/src/kisaniti_api/approval/`, migration 0009, `frontend/src/components/PlanReview.tsx` | 243 backend tests (concurrency, isolation, immutability), 321 contract checks, smoke through the website; report `…_slice3_approval.md` |
| Slice 4: contacts (masked, access log), consent events, three-language templates, instructions, preview channel, delivery records, queue (messaging/v1); pages: contacts, messages, instructions on the plan page | done; nothing is sent; translations unreviewed | `core/src/kisaniti_core/messaging.py`, `backend/src/kisaniti_api/messaging/`, migration 0010, `frontend/src/components/{Contacts,Instructions}Panel.tsx` | 265 backend tests, 563 core tests, 379 contract checks, smoke through the website; report `…_slice4_messaging.md` |
| Slice 5: actual sales consuming the reservation, off-plan reconciliation (provisional), voids, execution, report, outcomes, replies (sales/v1); pages: sales, execution and report on the plan page | done; reconciliation policy provisional | `backend/src/kisaniti_api/sales/`, migration 0011, `frontend/src/components/SalesPanel.tsx` | 295 backend tests, 563 core tests, 417 contract checks, smoke through the website; report `…_slice5_sales.md` |
| Slice 7: one-command chain `scripts/chain.py` (+ `chainlib.py` independent arithmetic), 13 steps, ledger sabotage run, CI stack job | done; fails on 3 of 4 planted bugs (a 1 g error is below a paisa and passes: recorded) | `scripts/chain.py`, `scripts/chainlib.py`, `backend/tests/test_chain_lib.py` (8 tests) | report `…_slice7_chain.md` |
| Research: forecasting evaluation kit (pairs, baselines, metrics, conformal, DM-HLN, Holm, bootstrap), synthetic tests only, no model run on prices | done; 8 open choices and a pair-count inconsistency await you (O70) | research repo `layers/L3a_forecasting/{evalkit,test_evalkit,count_eligible_pairs}.py`, `EVALKIT_ASSUMPTIONS.md` | 21 tests; 10 planted code changes caught |
| Research: parser B.1, index B.4, hand-check sheet, protocol draft, MSP table, Ladhar check | done | research repo | see OPEN_ITEMS, STATUS |
| Hardware build spec, hosting plan, demo plan, traceability rules | written | planning `docs/` | — |
| CI | 5 jobs (core, backend, ingestion, frontend, stack incl. the chain, its sabotaged run, and the smoke through the website with a fixture price release) | `.github/workflows/ci.yml` | check the latest run after the slice-1 push |

The complete step list with owners is [ROADMAP_ALL_STEPS.md](ROADMAP_ALL_STEPS.md); this ledger keeps only the queue of what is taken next.

## 2. Next, in order (take the first unchecked item)

- [x] **H-4a** device registry, assignments, tenant-scoped device endpoints, DB-backed registry for `kn-ingest` (CHAT-40, CHAT-41; report `…_H4a_device_registry.md`). Note: the placement valid at a reading's time is not yet computed anywhere; do it in H-5
- [x] **H-5** scheduled storage assessment, persisted hysteresis, placement at `measured_at`, endpoints, compose service `assessor` (CHAT-42; report `…_H5_storage_assessment.md`; demo `scripts/demo_storage.sh`)
- [x] **H-6** devices and storage page (both languages) (slice 1; report `…_slice1_foundation_data.md`)
- [x] **Slice 1** members, lots, stock ledger, mandi and MSP reference data, price import, storage page (CHAT-43)
- [x] **Slice 2** first plan (CHAT-44; report `…_slice2_first_plan.md`)
- [x] **Slice 3** review, approval with exact reservation, rejection, release (CHAT-45; report `…_slice3_approval.md`)
- [x] **Slice 4** contacts and consent, templates, preview channel, delivery records (CHAT-46; report `…_slice4_messaging.md`)
- [x] **Slice 5** replies, sales that consume the reservation, reconciliation, planned against actual (CHAT-47; report `…_slice5_sales.md`)
- [x] **Slice 7** the one-command chain (X.1; CHAT-48; report `…_slice7_chain.md`; `python3 scripts/chain.py`, `--sabotage ledger`)
- [ ] **Next: Slice 6** (it needs your approvals first, see below) or X.2 backtest. **Slice 6** (model comparison, hold-or-sell, live prices) needs your approvals first (protocol C.0, hand check). The real WhatsApp send waits for your Meta account (O.1, O.4)
- [ ] Logging and build identity (structured JSON logs, request ID end to end, commit hash in `/api/v1/meta` and footer) — see BUG_TRACEABILITY §2
- [ ] Allocation experiment decisions (solver time limit) then full seeds; forecasting only after the hand check and the frozen protocol
- [ ] D-layer hold-or-sell, L4 approval, L5 messages (preview channel first) — each after its contract slice is frozen

Blocked on the owner (2 Nov): sub-yard rule (O9), account/hosting (O34), Meta steps (O35), getting the parts (O71), Playwright decision. Done since this list was written: hand check and VALIDATED status (O1, O43), forecasting protocol frozen and run, message-text review.

## 2b. Hardware and final product: work order (2 Nov 2026)

Goal: the real node's readings appear on the website and feed the plan-to-sale chain, and the demo runs from the repos. Parts and architecture: [HARDWARE_FINAL_GUIDE.md](HARDWARE_FINAL_GUIDE.md), decision CHAT-50. Take the first unchecked item whose "needs" is met. **Part A does not wait for the parts.**

### A. Software, no parts needed (AI)

- [ ] **HW-S1** add Mosquitto and `kn-ingest` (`KN_REGISTRY=db`) to `compose.yaml` under a `hardware` profile, without changing the normal stack. Done when: the software node over MQTT and TLS reaches the website on the running stack, and a CI check covers it
- [ ] **HW-S2** provisioning script: creates the broker user and writes the git-ignored `secrets.h` from the certificate, so no value is hand-edited (Wi-Fi password is typed by the owner, never in chat). Done when: a firmware build with a generated file succeeds and the certificate check passes
- [ ] **HW-S3** `kn-ingest` reconnects if PostgreSQL restarts (known gap). Done when: a test restarts the database and no reading is lost
- [ ] **HW-S4** `scripts/hw_preflight.sh`: one command that checks the stack, broker, worker, certificate host against the current IP, device registration and the newest reading, printing PASS or FAIL per line
- [ ] **HW-S5** full rehearsal from a fresh clone with the software node: stack up → register → readings → assessment → lot shows the category → plan → approval → handover → sale. Record the result

### B. With the parts (BOTH: owner assembles, AI guides and connects)

- [ ] **HW-A** parts check: photo of everything and the probe wire label; AI confirms against CHAT-50 before power-on
- [ ] **HW-B** breadboard bench check (guide 5.1), then the same on the terminal board
- [ ] **HW-C** flash the real profile and connect (5.2–5.6): acks arrive and the reading is on the website
- [ ] **HW-D** contract check with `kn-conformance` (5.7): no FAIL; close BUG-009 on the real board
- [ ] **HW-E** salt calibration, settling time, outage tests (BUILD_GUIDE H4–H5), records in `firmware/records/`
- [ ] **HW-F** close the box, link to a lot, then the week run in grain (H6)

### C. Rest of the product and the final demo

- [ ] **P-1** hold-or-sell advice (not built; contract first, needs its parameter decisions)
- [ ] **P-2** hosting (needs the owner's account, O34) and the public-certificate check on the ESP32; then optionally Meta WhatsApp (O35, O60)
- [ ] **P-3** UI restyle with the owner's ideas
- [ ] **P-4** final demo rehearsal, at least twice on different days: node in grain at least 30 h earlier, guide §6 script, fallback ready. Any paper revision with what was really measured starts from the files of 2 Nov on branch `paper/submission-2026-11-02`, because `main` holds older text; `previous_versions/` stays local because that repo is public

### How chats work together

1. **Open each new chat in `~/Documents/kisaniti`**, not `sem_7/kisaniti`, and give the three repos access. GitHub is the shared memory: a chat knows only what is pushed or in these files.
2. **One item per chat.** Run two chats at the same time only if they touch different repos or different files.
3. **Start of a chat:** `git pull --rebase --autostash` in all three repos, read this ledger, state the item, its contract and its exclusions (`docs/blueprint/06_IMPLEMENTATION_TASK_TEMPLATE.md`).
4. **Contract first:** a change in behaviour, messages or schemas needs a decision and a frozen contract before code. Missing or UNKNOWN behaviour is reported, not guessed.
5. **Before an item is ticked:** its tests, the existing suites it touches, and `python3 scripts/chain.py` pass; CI is green after the push. A step that needs hardware counts only with real output saved in `firmware/records/`, never from a simulator.
6. **End of a chat:** commit only that chat's files (never `git add -A`), push, tick the item here with its evidence, and put new findings in OPEN_ITEMS and bugs in BUGS.md.
7. **All three GitHub repos are public** (checked 2 Nov 2026), so anything pushed can be read by anyone. **Never commit** `secrets.h`, `passwd`, `certs/`, `.env` or personal data, and never paste passwords in chat. Fictional farmers only.
8. **Integration gates:** after Part A (HW-S5 passes), after Part B (HW-F has real records) and before the demo (P-4). If a gate fails, fix it before going on.

Message to paste into a new chat:

> Continue KisanNiti. The three repos are side by side in ~/Documents: kisaniti, kisaniti_implementation, kisaniti_research_paper. Pull all three, then read kisaniti/AGENTS.md and kisaniti/docs/BUILD_LEDGER.md, including section 2b. Do item [ID] only, following its rules. Tell me first in five lines what the item needs. Verify with real output, never invent results, and tell me when something needs me. At the end, commit only your files, push, check CI and update the ledger.

## 3. Rules learned the hard way (do not rediscover)

- Contract first: a layer's code starts only after its slice is frozen (decision register, CHAT-28). Changes after a consumer exists are a new slice version.
- Never `git add -A` in the research repo. Commit only own files. Pull with `--rebase --autostash` before pushing.
- `uv` is **not installed** on this Mac; to change `uv.lock` run uv in Docker: `docker run --rm -v "$PWD":/repo -w /repo/backend -e UV_PROJECT_ENVIRONMENT=/tmp/venv python:3.13.9-slim-bookworm sh -c "pip install -q uv==0.9.5; uv lock"`, and copy the new wheels into the local venv by hand. Use the existing virtual environments with `PYTHONPATH` set (macOS hides `.pth` files from Python 3.13).
- Research scripts: `env/run <script>` in `kisaniti_research_paper`. Plain `python3 -I` for stdlib-only scripts.
- zsh does not split unquoted variables and does not take unquoted `--include=*.md`; macOS `sed -i ''`; no `timeout` command.
- FastAPI yield-dependencies commit after the response: commit inside the endpoint (`with tenant_scope(...)`).
- Instants must be `YYYY-MM-DDTHH:MM:SS.mmmZ` (use `kisaniti_core.timeutil.format_instant`).
- Every check that passes locally can still fail in CI (the paho/mypy case): mirror CI.
- The in-app browser pane must be visible (`show_pane preview`) while testing, or pages do not finish hydrating.
- Backend `mypy` locally needs `MYPYPATH=src:ingestion/src:../core/src`; the core's solver wrapper is excluded from unused-ignore checks in the backend config.
- Count examples and tests before writing a number in a commit message (three were wrong: count with `ls … | wc -l` first).
- A mutation run that shows no failure may simply not have applied: grep the file for the edit before believing it.
- Fictional data only. No real farmer data before the privacy controls and a legal review.

## 4. Commands that work

```bash
# test database (throwaway) on port 5434
docker run -d --name kn-test-pg -e POSTGRES_USER=kisaniti -e POSTGRES_PASSWORD=test-only -e POSTGRES_DB=kisaniti_test -p 5434:5432 postgres:17.6
# backend: tests, lint, types
cd ~/Documents/kisaniti_implementation/backend
export DATABASE_URL=postgresql+psycopg://kisaniti:test-only@localhost:5434/kisaniti_test PYTHONPATH=src:ingestion/src:../core/src
.venv/bin/python -m pytest ; .venv/bin/python -m ruff format . ; .venv/bin/python -m ruff check . ; .venv/bin/python -m mypy src tests
# ingestion
cd backend/ingestion && PYTHONPATH=src:../../core/src .venv/bin/python -m pytest
# contracts
cd ~/Documents/kisaniti_implementation && core/.venv/bin/python -m pytest contracts/tests -q
# frontend
cd frontend && npm run lint && npm run typecheck && npm run i18n:check && npm run build
# whole stack and smoke
docker compose up --build -d --wait ; scripts/smoke_access.sh ; scripts/smoke_access.sh http://localhost:3000
# import the cleaned prices and the MSP table into a running stack (real data; the release is git-ignored)
cd backend && DATABASE_URL=postgresql+psycopg://kisaniti:kisaniti-local-only@localhost:5433/kisaniti PYTHONPATH=src:ingestion/src:../core/src \
  .venv/bin/python -m kisaniti_api.market.import_release import-release --dir ~/Documents/kisaniti_research_paper/data/processed/basmati/release/basmati_punjab_release1
# same module: import-msp --csv <msp_schedule.csv>, set-status --dataset NAME --to VALIDATED --note "..." (only after the hand check)
# CI result after a push (one blocking call, no polling)
gh run watch $(gh run list --limit 1 --json databaseId -q '.[0].databaseId') --exit-status
```

Demo accounts in the running stack (fictional, local only): `demo.patiala` and `demo.ludhiana` with the passphrases in the last report; create more with `docker compose exec backend python -m kisaniti_api.access.cli create-fpo … --synthetic`.
