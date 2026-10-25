# Everything implemented, every bug traceable — the working rules

26 October 2026 • Requirement from the owner: everything is implemented, and any bug can be traced to its cause and fixed. This document sets the rules and keeps one list of every requirement with its honest status. It extends [DEMO_TEST_DEPLOY_PLAN.md](DEMO_TEST_DEPLOY_PLAN.md). Status words: **DONE** (built and tested, evidence linked), **PARTLY**, **PLANNED**, **BLOCKED** (needs a decision or an outside step).

## 1. Requirements and their status (the single list)

| # | What you asked for | Status 26 Oct | Evidence / next step |
|---|---|---|---|
| R1 | Paper published, claims only from real data | PARTLY | v2 revised; cleaning started; forecasting protocol drafted, not frozen, nothing run. Needs the hand check and your approval |
| R2 | Every choice justified: why this, why others compared | PARTLY | choice-and-evidence matrix in FINAL_DELIVERY_PLAN §2; most comparisons not yet run |
| R3 | Data cleaning validated | PARTLY | parser, 56-row hand-check sheet; the check itself is yours |
| R4 | Forecasting with ranges, honest negative results | PLANNED | pre-registered protocol (draft) |
| R5 | Hold-or-sell advice | PLANNED | no code yet |
| R6 | Fair allocation across members | PARTLY | core + oracle + validator done; policy comparison needs the basmati prices; paddy prices are constant |
| R7 | Storage sensing, hardware plugs in without rework | PARTLY | contract, firmware, simulator, conformance harness, PostgreSQL tables done; registry, assessment job and live page not built; real node not built |
| R8 | Login that separates FPOs | PARTLY | contract frozen; backend built and tested (82 tests; database wall proven independently); sign-in page and shell built ([report](../../kisaniti_implementation/docs/reports/2026-10-26_access_login_two_fpo.md)) |
| R9 | Website in Punjabi and English | PARTLY | sign-in, change password, home, users, status in both languages, checked by hand in the browser; Punjabi text unreviewed; other pages not built; no automated browser tests ([report](../../kisaniti_implementation/docs/reports/2026-10-26_signin_shell_punjabi.md)) |
| R10 | Manager review and approval with exact stock reservation | PLANNED | needs R8 first |
| R11 | WhatsApp messages in three languages | PLANNED | design done; preview channel then real channel; needs your Meta steps |
| R12 | A website that shows all of it, deployed where you can reach it | PARTLY | stack runs locally and is shown in the app; public deployment waits for your account decision |
| R13 | Everything tested, integrations checked | PARTLY | 5 CI jobs; live broker + PostgreSQL run; browser tests and whole-stack script not built |
| R14 | Bugs traceable and fixed | PARTLY | request IDs exist; bug register started with 5 real entries (`docs/BUGS.md`); structured logs, build identity and the screen-to-cause drill still to build |
| R15 | Hardware built to my specification, integrated later | PLANNED | [HARDWARE_BUILD_SPEC.md](HARDWARE_BUILD_SPEC.md) |

A requirement becomes DONE only when its row has: contract (if it crosses a boundary), code, tests, documentation, a recorded run, and a trace path (§2). The tracker's rule stands: no tick without an evidence link.

## 2. Tracing a bug from the screen to the cause

| Where | Trace key | Status |
|---|---|---|
| A request from the website | `X-Request-ID`, returned in every error body as `request_id` (EXISTS in the API) | EXISTS; the website must forward the ID to the API and show "Reference: <id>" in every error, in both languages (TO BUILD) |
| Server logs | one JSON line per request and per error with `request_id`, `user_id`, `fpo_id`, route, status, duration, build commit; no passwords, tokens, phone numbers or message text; a test fails if a secret pattern appears in a log | TO BUILD (standard library only) |
| Database | `audit_log` rows carry `request_id`, actor, FPO, event and the object IDs; sign-in, failures, resets, personal-data views and exports are always logged | TO BUILD with the login |
| Device path | `reading_id = device:boot:seq` end to end; rejected messages are stored raw with a hash; acknowledgements carry the same identity; `kn-report` and `kn-conformance` reproduce a device's history | EXISTS |
| Which build | `/api/v1/meta` and the log lines carry the commit hash and build time; the website footer shows it | TO BUILD |
| Experiments and paper numbers | every number from a script, result files with the commit and data manifests | EXISTS for the research repo; enforced by `numbers.py` before submission |

**Drill (to be run and recorded):** pick an error on the screen, take its reference, find the log line, then the audit row, then the code path, and write the three hops in the bug record. If any hop is missing, that is a bug in the traceability and is fixed first.

## 3. Bug lifecycle

1. **Record** it in `kisaniti_implementation/docs/BUGS.md`: ID, how found, trace key (request ID, reading ID or run ID), build commit, expected and actual result.
2. **Reproduce with a failing test first** (red). If it cannot be tested automatically, the record says why and what manual check replaces it.
3. **Fix** with the smallest change; the test goes green; the whole suite and CI stay green.
4. **Close** the record with root cause, fix commit and the regression test's name. A bug is never closed with "cannot reproduce" unless the trace shows why.
5. **Learn**: if the bug escaped because a check was missing, add that check (as happened with the live-broker bug and the CI mypy failure on 26 Oct, both recorded in the H-1 to H-3 report).

Severity and the rule for urgent fixes: anything that can show one FPO's data to another, lose a reading after its acknowledgement, or show invented numbers is **stop the line**: fixed before any other work, with a regression test, and the incident written up.

## 4. How "everything" is built without losing the thread

- Order stays contract first, then code, tests, documentation, paper section (the eight steps of the master plan).
- A layer is finished only when its row in §1 moves; I update this table at the end of every working session with the evidence, so you can see the real position at any time.
- New ideas go to OPEN_ITEMS as proposals; they do not enter the build until you approve them (architecture rule: never turn an UNKNOWN into behaviour).
- What I cannot do and you must (accounts, purchases, the Meta number, the hand check, supervisor decisions) stays on the list in OPEN_ITEMS with its owner, so nothing waits silently.
