# Quality and verification plan

15 October 2026 • Planning document. It says how we make sure **data is correct, code works, layers integrate and research claims hold**. The detailed acceptance cases already exist; this file ties them together and adds the cross-cutting engineering rules they did not cover. Implementation evidence goes to `kisaniti_implementation/docs/reports/` and `kisaniti_implementation/experiments/`.

Existing acceptance definitions (do not duplicate; extend):

- [Layer 1–3 acceptance](handoff/2026-10-14/KisanNiti_Layers_1_to_3_Acceptance.md) (T01–T48, evidence classes, completion gates)
- [Layer 4](handoff/2026-10-14/KisanNiti_Layer_4_Review_Proposal.md) (L4T01–22, L4I01–06)
- [Layer 5](communication/KisanNiti_Layer_5_Current.md) (L5T01–24)
- [WhatsApp](communication/WHATSAPP_MULTILINGUAL.md) (WA01–12)
- [Research plan](research/RESEARCH_AND_EXPERIMENT_PLAN.md) (EXP-00–14)

## 1. Four kinds of "correct"

| Kind | Question | How it is shown | Owner of evidence |
|---|---|---|---|
| **Data correct** | Does every stored value match its source, unit and meaning? | Source manifests, reconciliation audits, golden samples (§2) | Layer 1–2 reports |
| **Code correct** | Does each module do exactly what its contract says, including failures? | Unit, property and contract tests in CI (§3) | Every work-package report |
| **Integration correct** | Do the layers agree on IDs, units, versions, errors and side effects? | Contract tests on real producer/consumer, end-to-end scenarios on real PostgreSQL (§4) | W10, W12 reports |
| **Research correct** | Are comparisons fair and claims supported? | Experiment rules in the research plan; reproducibility checklist; peer review (§5) | `experiments/` reports |

A feature is "done" only when all four that apply have evidence. Status words are PASS / FAIL / NOT_RUN / BLOCKED.

## 2. Data correctness

1. **Source fidelity:** keep the original file or response with its checksum. Re-parsing the original must reproduce the normalised rows exactly (replay test, T03/T06).
2. **Reconciliation audit (each import):**
   - Rows in = rows stored + duplicates + rejected, exactly.
   - Per-series day counts are compared with the source's own listing where possible.
   - Report every mismatch.
3. **Golden sample (manual):**
   - Hand-check ~50 rows per import cycle against the official website: price, unit, date, mandi and variety.
   - Record the checker and date.
   - If any error is found, stop and fix the parser, then re-run.
4. **Unit sanity:**
   - Rupees per quintal stays per quintal.
   - Kilograms stay exact.
   - Every converted number keeps its original next to it.
5. **Variety and mandi mapping review:** every new name variant gets a human-reviewed mapping before use (OI-03).
6. **No fabricated data:**
   - Missing stays missing.
   - Synthetic rows carry `origin=SYNTHETIC` and are excluded from accuracy metrics by a query-level filter that is itself tested.
7. **Policy prices:** MSP values are copied from the official notification, with its URL, date and season. They are checked by a second person before use (CHAT-20).
8. **Sensor data:**
   - Salt calibration offset recorded.
   - Each reading keeps its device, boot, sequence and time quality.
   - Impossible values are rejected, never clipped.

## 3. Code correctness and engineering discipline

| Practice | Rule |
|---|---|
| Version control | Small commits with clear messages. One work package per branch; merge to `main` only after checks pass. Both repos on GitHub (private). |
| Continuous integration | GitHub Actions on every push: formatting, lint, type check, unit/property/contract tests, schema validation of `contracts/` examples. Database tests run against a real PostgreSQL service container. |
| Static checks | Python: type hints checked (e.g. mypy/pyright) and lint (e.g. ruff). TypeScript strict mode. Pin tool versions (OI-11). |
| Test types | Unit (exact arithmetic, validators, transport math, storage rule) → property/randomised (inventory never negative, no double reservation, fairness invariants) → contract (C01–C15, L4C, L5C) → integration (real DB) → end-to-end scenarios. |
| Optimizer | Brute-force oracle comparison on every change to the model; full objective tuple, not just ₹. |
| Coverage | Measure coverage, but judge by the listed acceptance cases, not a percentage. Every acceptance case ID maps to at least one automated test or an explicit manual procedure. |
| Review | Every merge has a written self-review against the task template. Use the `/code-review` and `/security-review` tools or a second person for inventory, approval, money, privacy and optimizer code. |
| Errors | No silent `except: pass`. Every failure has a named error state visible to the user or operator. |
| Secrets | Environment variables only. A secret-scanning check in CI. Never log phone numbers or tokens. |
| Migrations | Forward-only, reviewed, tested on a copy of real-shaped data; backup before applying. |

## 4. Integration correctness

1. **One definition per contract:**
   - Machine-readable schemas live in `kisaniti_implementation/contracts/`. Their meaning is defined in this planning folder.
   - Producer and consumer tests both load the same schema file.
2. **Contract change rule:** bump the version, update both sides, run all downstream tests before merging (Layer 4 §13).
3. **Golden end-to-end scenarios:** the Layer 1→3 scenario (acceptance §3) and the full workflow (Layer 5 §11) run in CI nightly with fixed clock and seeds.
4. **Failure injection:** provider outage, sensor offline, duplicate webhook, worker crash, concurrent approval — each has an automated test (T-, L4T-, L5T- IDs).
5. **Simulated vs real:** every integration report states which adapters were simulated; live checks (real data import, real ESP32, real WhatsApp) are separate gates.

## 5. Research correctness

- Follow the eight rules in the [research plan §0](research/RESEARCH_AND_EXPERIMENT_PLAN.md): leakage control, same targets, untouched test period, significance tests, reproducibility, negative results reported.
- Every comparison includes at least one simple baseline. A complex method must beat it significantly to be called better.
- Every design choice used in the paper has an entry in [design alternatives](research/DESIGN_ALTERNATIVES.md) saying why it was chosen, what was compared, and when the alternative would win.
- Paper numbers are copied from run reports by script, not typed by hand.

## 6. Documentation standard (needed for the paper)

Each component (adapter, cleaner, model, policy, optimizer, validator, UI flow, firmware) gets a short doc in the implementation repo with:

- purpose;
- contract version;
- inputs and outputs;
- the decision IDs it implements;
- alternatives considered (link to design alternatives);
- known limits;
- tests that cover it;
- latest evidence link.

Planning-side changes are recorded in the registers.

## 7. What can still go wrong (tracked)

Silent data drift at the source, model degradation over seasons, sensor drift, provider API changes, and user misunderstanding of advice. Mitigations:

- Monitor source freshness and gaps.
- Re-run forecast evaluation each season before re-promoting a model.
- Re-check sensor calibration each season.
- Use adapter contract tests with saved responses.
- Run comprehension tests (EXP-12).

None of these is a one-time pass.
