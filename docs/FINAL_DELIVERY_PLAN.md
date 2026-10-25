# Final delivery plan — publishable paper, working product, plug-in hardware

26 October 2026 • Written at the owner's request. It states what "done" means for three deliverables that must stay consistent: the paper, the software, and the hardware. It adds no new technology. It extends [NEW_CHAT_START.md](NEW_CHAT_START.md) and the [tracker](PROJECT_TRACKER.md); unsettled findings are in [OPEN_ITEMS.md](OPEN_ITEMS.md). Classification used below: **EXISTS** (built and tested), **TO BUILD**, **DECISION** (needs the owner), **UNKNOWN** (no evidence yet; not to be turned into behaviour).

## 1. Three rules that tie everything together

1. **Every component has a recorded "why this, why not the others".** The paper's argument is a table: for each design choice, what was chosen, which alternatives were run on the same data, what the evidence says, and what the limit is. A result nobody compared is a claim without support, so it is reported as an assumption.
2. **Contract first, then code, then evidence.** A layer's schema is frozen before its code (CHAT-28). Anything that talks across a boundary (hardware, WhatsApp, price feeds) does so through a versioned contract in `kisaniti_implementation/contracts/`, never through shared internals.
3. **The claim ceiling is set by the data, not by the story.** With five basmati seasons, only large forecasting gains are detectable (about 11–17%); hold-or-sell rests on five seasons; hardware results are simulated until the kit runs. Whatever the data cannot support is reported as a limitation or future work, in the paper and in the product's wording.

## 2. The paper: how every choice is justified

### 2.1 Choice-and-evidence matrix (the spine of the Methods and Results sections)

| Component | Chosen | Alternatives that are run | Evidence state | Where |
|---|---|---|---|---|
| Cleaning | flag, never delete; blank-date rule if confirmed | raw only; + blank-date rule; + outlier flags as exclusions; MAD vs Hampel vs IQR | Parser, reconciliation and tests done. Methods not yet compared; blank-date rule awaits the hand check | tracker B.2–B.3 |
| Variety index | median modal price, ≥ 3 mandis, no filling | with/without blank-date rows; sub-yards merged or not; min mandis 2/3/4 | Built both ways; sensitivity measured for sub-yards (about 20% of days change) | B.4 |
| Forecasting | decided by the pre-registered test, possibly "baseline is enough" | naive, seasonal naive, moving average, ETS, ARIMA, LightGBM, GRU, Chronos-2 | Protocol drafted, not frozen, nothing run | C.0–C.7 |
| Prediction ranges | the method with the best checked coverage | analytic, quantile, split-conformal, Chronos quantiles | Not run | C.5 |
| Storage rule | 24 h median + hysteresis, in-grain probe | latest reading; shed air; (rule alternatives) | Simulated scenarios done (≤ 0.7 category changes/week vs 14–27); real node not built | L3b |
| Hold-or-sell | rule with storage loss, holding cost, interest | sell at harvest; hold to a fixed month; median-forecast rule; perfect foresight | Not started; five seasons only, exploratory | D.1–D.4 |
| Allocation and fairness | exact lexicographic MILP with leximin | nearest mandi; independent choice; weighted sums; max-min only; utilitarian; greedy; four solvers; brute-force oracle | Core, oracle, validator done; policy comparison running | E.1–E.5 |
| Whole system | the full chain | simple practice (sell at harvest to nearest mandi), per season and scenario | Not started | I.2 |
| Hardware | ESP32 + SHT30 in the grain | DHT22 and others (desk comparison, datasheets); Wokwi twin versus real node | Desk comparison done; real node waits for the kit | F |

"Why others compared" is also answered at the related-work level: the 57 references plus the cleaning papers explain what existing studies did. The gap statement must be corrected first (Ladhar et al. 2023 contradicts gap 4 as written; see OPEN_ITEMS O14).

### 2.2 What makes the paper hard to reject (checklist, each item needs recorded evidence)

- Pre-registered forecasting protocol frozen before the test season is touched; negative results reported; claims limited by the power check.
- Cleaning validated by the 56-row hand check and one independent spot check; every number from a script (`paper/numbers.py`).
- Every reference opened and the claim checked; abstract-only sources not used as evidence.
- Reproducibility package: one command per result, pinned environment, data manifests with checksums, archived with a DOI (tracker I.4).
- Honest limits: modal price is not the farmer's price; one source (AGMARKNET) with spot checks; a probe does not detect spoilage; synthetic farmers are labelled synthetic; no user study.
- Authors' own wording; AI assistance declared per Springer policy; similarity check re-run (H.4, H.6).
- Venue, template and page limit confirmed with the supervisor (**DECISION**, H.7). The plan assumes about 11 pages, so the full protocol, solver tables and hardware build details go into an archived supplement.

### 2.3 One-paper scope (CHAT-33)

The paper's story is: *a pipeline whose every stage was compared with simpler alternatives on real data, with honest uncertainty, plus a hardware-ready storage layer.* Contributions that survive even if no forecasting model beats the baseline: the reconciled cleaned dataset and its validation, the fair allocation with price of fairness, the storage rule's stability result, and the protocol itself.

## 3. The product: what "working and documented" means

- **One command** starts everything (Docker Compose); CI green on real PostgreSQL (EXISTS for the skeleton; TO BUILD for the layers).
- **Flow on real price data:** data, cleaning, forecast with ranges, storage risk, hold-or-sell, fair plan, manager approval (atomic reservation), WhatsApp test message, sale from a J-form, stock update (tracker I.1).
- **Built layer by layer** with the eight steps of the master plan, so each layer ships with its contract, tests, guide and paper section.
- **Wording rules** in the UI follow the claim ceiling: ranges, "no reliable signal" beyond 28 days (OPEN_ITEMS O3), storage category not spoilage.
- **Documentation set:** layer guides (exist for L3b), user manual, deployment guide, hardware build guide (exists), reproducibility guide for every paper result.

## 4. Hardware that plugs in without rework

### 4.1 What already makes this easy (EXISTS)

| Piece | Where | Why it helps |
|---|---|---|
| Versioned JSON schemas for telemetry, ack, device assignment, reading, assessment | `contracts/sensor/v1/` with valid and invalid examples | A device is "compatible" if it passes the schema; no code sharing needed |
| MQTT over TLS, QoS 1, application ack after the database write | `backend/ingestion/`, firmware | A broker acknowledgement does not prove storage; duplicates are handled by the `(device, boot, seq)` key |
| `source_origin` on every message (for example `WOKWI_TWIN`) | firmware, schemas | Real, simulated and twin data stay separable, so simulated data is never counted as evidence of accuracy |
| Storage rule in the shared `core/` | `core/src/kisaniti_core/storage/` (31 tests) | Same function for experiments and product; the device only supplies readings |
| Firmware with native tests, real and Wokwi profiles | `firmware/` | The same code path runs in the twin and on the board |
| Wokwi twin with an SHT3x chip model | `firmware/wokwi/` | Anyone can run the node without hardware |

### 4.2 The integration rule

> **The backend never knows what is on the other end.** It sees only contract messages. A real ESP32, the Wokwi twin, a software simulator and any future device are interchangeable if they pass the same conformance test.

### 4.3 What is still to build (TO BUILD, in this order)

| # | Step | Done when |
|---|---|---|
| H-1 | **Device conformance test:** one script that connects any publisher to the broker, sends valid, duplicate, out-of-order and malformed messages, and checks acks and rejections | The firmware profile in the twin, and the software simulator, both pass; the report is stored |
| H-2 | **Software simulator** that speaks the same contract with a configurable scenario (humid, dry, outage, sensor fault), flagged `source_origin=SIMULATOR` | Runs in CI, so the whole pipeline is tested with no board |
| H-3 | **Persist readings in PostgreSQL through the API** (migration, idempotent insert, per-device and per-lot queries) | Concurrency and idempotency tests on real PostgreSQL |
| H-4 | **Device registry and assignment** (device to lot to organisation, tenant-isolated) using the existing `device_assignment` contract; per-device MQTT credentials | A device that is not assigned is rejected; multi-FPO isolation test passes |
| H-5 | **Scheduled storage assessment with stored hysteresis state** feeding the planner and the hold-or-sell rule | Survives a service restart; same output as the research simulation on the same readings |
| H-6 | **Live view and alerts in the app** (latest reading, category, last-seen, outage) | Works from the simulator today and the real node later without change |
| H-7 | **Firmware hardening for the real node:** flash queue instead of the RAM buffer (the stated v1 limit), time sync, over-the-air update decision (**DECISION**) | Outage test from the build guide passes |
| H-8 | **Real node acceptance** (BUILD_GUIDE H4–H7): salt calibration, settling time, outage test, one-week run in a labelled grain container | Bench records in `firmware/records/`; results written up without overstating |
| H-9 | **Extension note:** how to add a second sensor type (for example a weighbridge or a gate scale): new schema version under `contracts/`, new ingestion adapter, same conformance test | A one-page guide plus a worked example |

Steps H-1 to H-6 need no hardware and can start now; H-8 waits for the kit (tracker F.3).

### 4.4 Rules that keep it connectable

- Contract changes are versioned (`v1`, `v2`), never edited in place.
- No device-specific logic in the backend; differences go into the schema or an adapter.
- Every stored reading carries device, boot, sequence number, time, and `source_origin`.
- Failure is explicit: a missing sensor yields `UNKNOWN`, not a guess (EXISTS in the storage rule).

## 5. Order of work

Critical path for the paper: **hand check → cleaning gate → protocol frozen → forecasting → hold-or-sell → whole-system backtest → paper assembly → supervisor review.** Allocation, storage and the product run alongside it.

| Stage | Steps | Needs from the owner | Runs in parallel with |
|---|---|---|---|
| 1. Unblock | O1 hand check; approve and freeze C.0; choose sub-yard rule; correct the literature gap (O14) | the hand check (about an hour), three decisions | E, H-1 to H-3 |
| 2. Cleaning gate | B.3 candidate screens scored on the hand-checked sample; B.4 final index; B.5 dataset release | — | E, H-4 to H-6 |
| 3. Forecasting | C.1–C.7 on the frozen protocol | — | E.5, G.1 contract freezes |
| 4. Decision layers | D.1–D.4; G.3–G.5 screens; plan generation and approval | storage and holding-cost assumptions confirmed | H-5 |
| 5. Hardware | F.3 kit purchase early; H-7, H-8 | buy the kit; run the bench tests | G.6 |
| 6. Integration | I.1 one-command run; I.2 whole-system backtest; I.3 demo from a fresh clone | WhatsApp test number | — |
| 7. Paper | H.3 sections from results; H.4 authors rewrite; H.5 references; H.6 checker; H.8 declarations; H.9 final numbers and supervisor review | supervisor review, venue | — |

Order the hardware kit now: it is the longest wait that needs no other result.

## 6. Gates (stop and decide)

G1 after cleaning (hand check agrees); G2 before forecasting (frozen protocol, no long history by default); G3 after forecasting (baseline is hard to beat is a valid result); G4 before writing results (every number from a script); **G5 before integration claims** (the whole-system run uses only contract messages from real, simulated or twin sources, each labelled); G6 before submission (supervisor, checker score, references, declarations, venue rules). Details of G1–G4, G6 are in NEW_CHAT_START.md.

## 7. Risks

| Risk | Consequence | Response |
|---|---|---|
| No model beats the baseline | Forecasting section is a negative result | Say so; the pipeline, fairness and protocol carry the paper |
| Hand check rejects the blank-date rule | About 9.5% fewer rows; thinner early seasons | Use dated-only as primary (rule already fixed in the protocol) |
| Kit delayed or the bench test fails | No real-node results | Paper uses the twin and simulations and says so; real results added if collected in time |
| Contract drift between firmware and backend | Silent data loss | Conformance test (H-1) in CI on every change |
| Scope creep (more crops, regions, models) | Nothing finishes | New ideas go to OPEN_ITEMS as future work unless the owner changes scope |

## 8. Decisions needed from the owner

1. Approve and freeze the C.0 protocol (OPEN_ITEMS O2, O18–O22).
2. The sub-yard merge rule (O9) and the demo's MSP season (O27).
3. Target venue, deadline and page limit (H.7); this fixes the schedule, which this plan deliberately leaves without dates.
4. Whether the first real node uses over-the-air updates (H-7).
5. Buy the kit (F.3).
