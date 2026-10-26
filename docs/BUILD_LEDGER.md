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
| Research: parser B.1, index B.4, hand-check sheet, protocol draft, MSP table, Ladhar check | done | research repo | see OPEN_ITEMS, STATUS |
| Hardware build spec, hosting plan, demo plan, traceability rules | written | planning `docs/` | — |
| CI | 5 jobs (core, backend, ingestion, frontend, stack incl. smoke through the website with a fixture price release) | `.github/workflows/ci.yml` | check the latest run after the slice-1 push |

The complete step list with owners is [ROADMAP_ALL_STEPS.md](ROADMAP_ALL_STEPS.md); this ledger keeps only the queue of what is taken next.

## 2. Next, in order (take the first unchecked item)

- [x] **H-4a** device registry, assignments, tenant-scoped device endpoints, DB-backed registry for `kn-ingest` (CHAT-40, CHAT-41; report `…_H4a_device_registry.md`). Note: the placement valid at a reading's time is not yet computed anywhere; do it in H-5
- [x] **H-5** scheduled storage assessment, persisted hysteresis, placement at `measured_at`, endpoints, compose service `assessor` (CHAT-42; report `…_H5_storage_assessment.md`; demo `scripts/demo_storage.sh`)
- [x] **H-6** devices and storage page (both languages) (slice 1; report `…_slice1_foundation_data.md`)
- [x] **Slice 1** members, lots, stock ledger, mandi and MSP reference data, price import, storage page (CHAT-43)
- [x] **Slice 2** first plan (CHAT-44; report `…_slice2_first_plan.md`)
- [x] **Slice 3** review, approval with exact reservation, rejection, release (CHAT-45; report `…_slice3_approval.md`)
- [x] **Slice 4** contacts and consent, templates, preview channel, delivery records (CHAT-46; report `…_slice4_messaging.md`)
- [ ] **Slice 5 (next)** replies, actual sales that consume the reservation (RESERVATION_CONSUMED and SOLD in one transaction), reconciliation, planned against actual (S5.1 to S5.7). Contract slice first (CHAT-28); needs your approval of the scope. The real WhatsApp send waits for your Meta account (O.1, O.4)
- [ ] Logging and build identity (structured JSON logs, request ID end to end, commit hash in `/api/v1/meta` and footer) — see BUG_TRACEABILITY §2
- [ ] Allocation experiment decisions (solver time limit) then full seeds; forecasting only after the hand check and the frozen protocol
- [ ] D-layer hold-or-sell, L4 approval, L5 messages (preview channel first) — each after its contract slice is frozen

Blocked on the owner: 56-row hand check (O1; also gates the price dataset's VALIDATED status, O43), protocol approval (O2), sub-yard rule (O9), account/hosting (O34), Meta steps (O35), kit purchase, Punjabi text review, Playwright decision.

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
