# Build ledger — where the work stands, so nothing is done twice

Read this first in a new session, after `NEW_CHAT_START.md`. Updated at the end of every working step. Repos are in `~/Documents` (`kisaniti`, `kisaniti_implementation`, `kisaniti_research_paper`); the session folder `sem_7/kisaniti` is an old copy and must not be used. The owner will change the UI later: do not polish pages; build function and tests.

## 1. Done (with evidence)

| Area | State | Where | Evidence |
|---|---|---|---|
| Shared contract, FPO table | frozen, built | product `contracts/shared/v1`, migration 0001 | CI |
| L3b sensor path: contract, firmware, `core.storage`, MQTT ingestion | built, tested | product `firmware/`, `backend/ingestion/`, `core/` | 45 ingestion tests; live broker script |
| H-1 conformance harness, H-2 software node, H-3 PostgreSQL readings | done | `backend/ingestion/src/kisaniti_ingestion/{simulator,conformance}.py`, migration 0002 | report `docs/reports/2026-10-26_H1-H3_*` |
| Access slice (login, sessions, roles, two walls) | done, contract `access/v1` frozen + amendment A1 | `backend/src/kisaniti_api/access/`, migration 0003 | 82 backend tests; report `…_access_login_two_fpo.md` |
| Website: sign-in, change password, home, users, status; Punjabi and English; API proxy | done (UI to be restyled by the owner) | `frontend/src/` | report `…_signin_shell_punjabi.md` |
| Research: parser B.1, index B.4, hand-check sheet, protocol draft, MSP table, Ladhar check | done | research repo | see OPEN_ITEMS, STATUS |
| Hardware build spec, hosting plan, demo plan, traceability rules | written | planning `docs/` | — |
| CI | 5 jobs green (core, backend, ingestion, frontend, stack incl. smoke) | `.github/workflows/ci.yml` | last green run after commit `da84e5a` |

## 2. Next, in order (take the first unchecked item)

- [ ] **H-4a** device registry, assignments, FPO resolved at `measured_at`, tenant-scoped device endpoints, DB-backed registry for `kn-ingest` (decision CHAT-40)
- [ ] H-5 scheduled storage assessment with persisted hysteresis (`core.storage` + readings)
- [ ] H-6 devices and storage page (both languages) from simulator data
- [ ] Logging and build identity (structured JSON logs, request ID end to end, commit hash in `/api/v1/meta` and footer) — see BUG_TRACEABILITY §2
- [ ] Market data page D1 (needs an L1/L2 contract slice first — freeze before code)
- [ ] Allocation experiment decisions (solver time limit) then full seeds; forecasting only after the hand check and the frozen protocol
- [ ] D-layer hold-or-sell, L4 approval, L5 messages (preview channel first) — each after its contract slice is frozen

Blocked on the owner: 56-row hand check (O1), protocol approval (O2), sub-yard rule (O9), account/hosting (O34), Meta steps (O35), kit purchase, Punjabi text review, Playwright decision.

## 3. Rules learned the hard way (do not rediscover)

- Contract first: a layer's code starts only after its slice is frozen (decision register, CHAT-28). Changes after a consumer exists are a new slice version.
- Never `git add -A` in the research repo. Commit only own files. Pull with `--rebase --autostash` before pushing.
- `uv` is **not installed** on this Mac. Use the existing virtual environments with `PYTHONPATH` set (macOS hides `.pth` files from Python 3.13).
- Research scripts: `env/run <script>` in `kisaniti_research_paper`. Plain `python3 -I` for stdlib-only scripts.
- zsh does not split unquoted variables and does not take unquoted `--include=*.md`; macOS `sed -i ''`; no `timeout` command.
- FastAPI yield-dependencies commit after the response: commit inside the endpoint (`with tenant_scope(...)`).
- Instants must be `YYYY-MM-DDTHH:MM:SS.mmmZ` (use `kisaniti_core.timeutil.format_instant`).
- Every check that passes locally can still fail in CI (the paho/mypy case): mirror CI.
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
# CI result after a push (one blocking call, no polling)
gh run watch $(gh run list --limit 1 --json databaseId -q '.[0].databaseId') --exit-status
```

Demo accounts in the running stack (fictional, local only): `demo.patiala` and `demo.ludhiana` with the passphrases in the last report; create more with `docker compose exec backend python -m kisaniti_api.access.cli create-fpo … --synthetic`.
