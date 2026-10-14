# KisanNiti — Layer 1–3 integration and acceptance plan

Revision 1 • 14 October 2026 • Companion to the [integrated handoff](KisanNiti_Layers_1_to_3_Handoff.md).

**All application tests below are NOT RUN in this review.** This file defines acceptance requirements; it is not an automated test suite or a passing test report. Tests that depend on a policy not yet selected are also blocked on that policy. The review's documentation checks are recorded separately.

## 1. What counts as tested

| Evidence class | What it establishes | What it does not establish |
|---|---|---|
| Document consistency | References, fixture arithmetic, explicit contracts and test coverage in the handoff | Implemented behavior |
| Unit/domain | Local exact rules and failure handling for specified cases | Real database/provider/device behavior |
| Contract | Producer/consumer agree on schema, units, status, identity and version | Availability or truth of the real source |
| Integration | Actual application modules and required persistence/transport work together | Forecast usefulness or field science |
| Forecast evaluation | Measured held-out errors/coverage for the recorded dataset/model | Guaranteed future prices or all-crop support |
| Optimizer oracle | Agreement with exhaustive exact reference on small cases | Performance on every large FPO |
| Live-provider | Authorized real adapter request/import works with recorded version | Complete nationwide history |
| Hardware bench | Physical device measurements reach the backend correctly | Whole-shed coverage or validated spoilage prediction |
| Field/pilot | Behavior tested in the recorded real operational context | Zero possible failures elsewhere |

Record code revision, schema/policy/model/firmware versions, fixture seed/data checksum, environment, command or procedure, expected result, observed result, elapsed time, evidence path and reviewer. Status is PASS / FAIL / NOT_RUN / BLOCKED. A skipped test is not PASS. Never leave the table at PASS after changing the tested contract without assessing/running the affected checks.

## 2. Traceable acceptance cases

Contract IDs C01–C15 and decision IDs D01–D20 are defined in the handoff. Every case below starts NOT_RUN.

| ID | Contracts | Kind / scenario | Required observation |
|---|---|---|---|
| T01 | C01,C02,C04 | Fixtures: same seed/config run twice | Same logical dataset/checksums; counts exactly 5/1,000/3,000 and 172,800 baseline sensor events; faults separate |
| T02 | C01,C14 | Real market sample import | Authorized access, actual units/schema/coverage, allowed-use notes, raw evidence and measured manifest; missing access reported as blocked |
| T03 | C01,C03,C15 | Replay identical market batch | No duplicate canonical observation or downstream forecast job for unchanged inputs |
| T04 | C01,C03,C14 | Corrected report and conflicting report | Proven correction links old/new versions; unresolved conflict remains review; last receipt alone does not win |
| T05 | C03,C14 | AGMARKNET row republished by another source | Retain provenance without counting twice or silently averaging |
| T06 | C01,C14,C15 | Interrupted archive job then resume | Resume from checkpoint; no silent missing partition or double import; counts reconcile |
| T07 | C02,C05 | Cross-FPO actor/reference manipulation | Reject unauthorized reads/writes/joins/exports; global mandi catalogue does not bypass isolation |
| T08 | C01,C03,C05 | Units, dates and numeric parsing | 2,400 INR/quintal becomes exactly 24 INR/kg under selected normalization; unknown bag weight/ambiguous dates/unsupported precision explicitly fail affected use |
| T09 | C03,C05 | Missing/zero/contradictory market fields | Missing not zero; inconsistent min/modal/max quarantined; optional missing arrivals compatible with price-only model |
| T10 | C03,C05,C14 | Genuine spike, duplicate date, missing day | Statistical flag preserves otherwise valid spike; legitimate series distinctions survive; no fabricated observed day |
| T11 | C03,C05 | Past-only anomaly review | No future rows enter screen; MAD zero/short history unassessable; real reviewed examples measure false alarms separately from synthetic faults |
| T12 | C02,C05 | Exact inventory sequences | Derived availability matches sold + remaining reserved + adjustments; no float drift, negative stock or duplicate effect |
| T13 | C02,C04,C09 | Device moved between locations | Old samples resolve old effective assignment; ambiguous timestamp/assignment cannot silently use current location |
| T14 | C04,C05 | Simulator vs hardware adapter contract | Same required schema/unit/status semantics; origin differs and remains visible; invalid versions rejected explicitly |
| T15 | C04,C05 | Duplicate, reordered, missing-channel and invalid sensor events | One canonical effect per event; bad/missing channel never becomes plausible zero; sequence gaps visible |
| T16 | C04,C09 | Time, stale, delayed and future-clock readings | Measurement age is distinct from receipt age; stale/invalid/time-unknown produces UNKNOWN under policy; boundary behavior tested |
| T17 | C04,C09 | Sensor credentials and forged tenant/location/origin | Registered source authority wins; unauthorized device topics/FPO bindings rejected and audited |
| T18 | C04,C15 | MQTT reconnect / receiver crash after receipt | Defined durable acceptance/ack semantics survive retry; no false persisted-data claim or duplicate assessment effect |
| T19 | C04,C09 | Physical ESP32 + SHT30 bench | Firmware compiles; delivered pin labels checked; values/event IDs match logs and stored records; chip read errors visible |
| T20 | C04,C09,C15 | Physical power loss, Wi-Fi loss, clock loss, reboot and buffer exhaustion | No invented samples/freshness; replay retains identity/time; lost history and buffer limits explicitly reported; recovery follows documented policy |
| T21 | C09,C11 | Unplugged or never-installed hardware | Sensor health/assessment becomes unavailable/UNKNOWN; otherwise admissible plan remains possible; no fake LOW |
| T22 | C09,C11 | Experimental risk policy and synthetic categories | Every boundary/duration/hysteresis case reproducible; policy/evidence/source-origin label retained; unsupported crops UNKNOWN; science claim separate |
| T23 | C06,C07,C05 | Route unavailable, wrong origin, expired/missing quote | Affected choices excluded with evidence reason; no zero distance/price fallback or guessed validity |
| T24 | C07,C12 | Fixed-trip quote, load limits and shared cost | Exact charge matches declared applicability; one active shared trip charged once if sharing enabled; unsupported sharing blocked |
| T25 | C08,C14 | Forecast leakage and chronological splits | Features/transforms/graph fitted within training fold; future prices/arrivals absent; untouched test and pretrained-overlap limits recorded |
| T26 | C08,C14 | Forecast comparisons | Same origins/targets/horizons and unit; MAE by series/horizon plus coverage/spikes/latency; supported-series registry matches measured evaluation |
| T27 | C05,C08 | Forecast contract/failure | Model/input/numeric versions present; invalid price/interval/applicability unavailable; unavailable never price zero |
| T28 | C03,C08,C15 | Many changed observations in one batch | Coalesced inference for changed dependent contexts; unchanged inputs skipped; no per-row training; failed job recoverable |
| T29 | C10,C11 | Frozen optimizer input | Only authorized selected lots/markets/dates; exact quantities; policies present; no database/network/provider objects |
| T30 | C11,C12 | Tiny exhaustive oracle | Complete business objective tuple matches proven solver result, not merely total net return |
| T31 | C11,C12 | Per-farmer fairness with multiple lots | Aggregate each farmer once; entire sorted fulfilment vector optimized; no extra weight for multiple lots or arbitrary named bottleneck |
| T32 | C11,C12 | Full/partial and incompatible increments | Correct exact remainders; full-mode incompatibility explained; no silent downgrade; actual-sale precision remains independent |
| T33 | C09,C11,C12 | Storage hierarchy and economic guardrail | Higher optima preserved; HIGH then MEDIUM timing per policy; UNKNOWN no fabricated rank; unconfigured production guardrail blocks claimed readiness |
| T34 | C11,C12 | MarketLoadPolicy variants | DISABLED no effect; ADVISORY no feasibility/objective effect; synthetic STRICT checked; missing strict evidence not zero/infinity |
| T35 | C11,C12 | Numerical limits and reconstruction | Exact integer decisions reconstructed and independently checked; overflow/invalid scale/unsafe coefficient is technical failure, not business infeasibility |
| T36 | C12,C13 | Timeout, crash, infeasibility and zero allocation | Separate termination/proof/allocation axes; certified objective prefix accurate; no unproven optimality or implied approval eligibility |
| T37 | C11,C12 | Repeatability and performance | Same fully solved input gives canonical tied result; record build/solve/validation time, memory, dimensions/stages; no hidden pruning |
| T38 | C10,C12,C13 | Independent output validation and edited/manual revision | Reject over-allocation or arithmetic errors independently of solver; shared hard rules; old revision unchanged; manual-only evaluation has no fabricated OptimizationRun |
| T39 | C01,C05,C10,C13 | Complete Layer 1→2→3 path on actual DB | Input → usable facts → derivations → frozen run → independently validated proposal; no stock reservation or external instruction |
| T40 | C10,C13,C15 | Later price/sensor/stock update | Existing revision remains unchanged; changed evidence visible; explicit new run/revision uses current input |
| T41 | C10,C11,C12,C14 | Snapshot replay and model rollback | Reconstruct original inputs/evaluation from versions; rollback model tagged; replay does not become a fresh observation or new stock effect |
| T42 | C01,C10,C14 | Demo/real separation | Fictional identities never receive real messages; synthetic rows excluded from real training/evaluation; mixed-source demo labelled |
| T43 | C02,C13 | Layer 4 duplicate/concurrent approvals | Current-stock transaction prevents double reservation; approval plus all reservations atomic; provider calls outside stock locks |
| T44 | C02,C13 | Layer 5 linked partial sale/cancellation/replan | Consume corresponding reservation exactly once; release only unexecuted quantity; approved history remains; new plan sees correct current availability |
| T45 | C02,C13 | Off-plan sale with active reservations | BLOCKED on agreed reconciliation workflow; do not guess reservation release order, reject reality merely because plan exists, or claim success after failed stock update |
| T46 | C13,C15 | Later communication failure/retry | Only approved assignments source instructions; test destination/sandbox; failed delivery does not undo approval or reserve/sell again |
| T47 | C01,C10,C14,C15 | Backup/restore, schema change and repair | Restore recorded operational snapshot, replay safely, recover source jobs and retain audit lineage; required incompatible schema fails visibly |
| T48 | C01,C04,C10,C15 | Logs and operational diagnosis | Correlate batch/job/device/run; expose stale sources and queue failures; no secrets or personal farmer records in public logs |

## 3. First connected scenario — Layers 1–3 only

Use a small explicit DEMO profile before scale testing:

1. Create two fictional FPOs so isolation is testable. Use one authorized FPO for the planning request.
2. Register one farmer, one 1,000 kg wheat lot, one farm-shed placement and one simulated registered device. No active reservation or sold stock initially.
3. Declare a synthetic planning increment of 1,000 kg for this one-lot fixture only; it does not select the production increment. Import a permitted real market sample, or a visibly synthetic source fixture if access is blocked. Run the same parser/cleaning contracts. A synthetic sample does not establish forecasting accuracy.
4. For deterministic integration, use a **forecast adapter fixture** for two eligible candidate mandis on one explicitly allowed future date: A = 24 INR/kg, B = 25 INR/kg. This is not a model prediction claim.
5. Provide applicable route fixtures and a dedicated-trip fixture quote for each candidate: A = 1,000 INR, B = 3,000 INR, each covering up to the full 1,000 kg on that date. Each synthetic quote has a positive-load range up to 1,000 kg, one dedicated trip available for its candidate, no other modeled charges and an explicit validity interval covering the scenario. Each charge activates only when its candidate is used. The fixture increment prevents splitting this lot; all higher priorities and economic admissibility are explicitly equal.
6. Use UNKNOWN storage for the real-evidence path until a policy is selected. Separately test synthetic HIGH/MEDIUM categories without calling them measured spoilage.
7. Freeze the input, solve, run PlanValidator and save a demo proposal. A gives gross 24,000 and expected net selling return 23,000; B gives gross 25,000 and expected net selling return 22,000. The expected full-load result is A under the declared fixture conditions.
8. Verify allocated = 1,000, remainder = 0 and **available stock remains 1,000**, because this is only a proposal.
9. Send a duplicate sensor event and re-import the same price record. Verify no duplicate canonical effect, no training and no overwritten proposal.
10. Disconnect the sensor; the storage assessment becomes UNKNOWN with a reason. This alone must not invalidate the otherwise usable plan.
11. Fail the required route/quote for every candidate in a new request. Report missing usable evidence; do not produce a zero-cost plan or claim proven physical impossibility.

Encode these fixture transport details and exact dates/IDs in W01; this narrative is the expected behavior, not an executable fixture. Replace adapters with real providers one at a time and rerun the same contract/integration scenarios with origin labels and real-source evidence.

## 4. Exact inventory example for the later full workflow

This illustrates stable linked-sale arithmetic. Actor authority and freshness rules still require Layer 4–5 contracts.

| Step | Actual sold | Remaining reservation | Available from 1,000 kg |
|---|---:|---:|---:|
| Register lot | 0 | 0 | 1,000 |
| Generate 600 kg proposal | 0 | 0 | 1,000 |
| Approve/reserve 600 kg | 0 | 600 | 400 |
| Record linked actual sale of 250 kg | 250 | 350 | 400 |
| Repeat the same sale request | 250 | 350 | 400 |
| Release remaining 350 kg commitment | 250 | 0 | 750 |
| New planning cycle | 250 | 0 | Up to 750 available |

The linked sale converts reserved stock into sold stock. It does not subtract 250 a second time from already-free stock. Cancellation does not erase historical approval or the 250 kg actually sold. Off-plan reconciliation is a separate unresolved workflow and cannot be inferred from this example.

## 5. Model and optimizer acceptance reports

Forecast report must include requested versus usable series, train/validation/test dates, raw versus usable counts, first-known/revision limitations, common-target errors, per-horizon errors, abstentions, intervals if used, runtime/memory and exact artifacts/configuration. No fixed percentage improvement is promised before measurement. Record why a baseline or more complex model was selected.

Optimizer report must include exact input identity, farmers/lots/markets/dates/admissible choices, increment scales, policy versions, candidate constraints, objective-stage outcomes/proofs, numerical tolerances, build/solve/validation time, memory and termination. Compare complete exact objective tuples on tiny enumerated cases. PlanValidator checks feasibility/arithmetic independently; it does not prove fairness optimality or global optimality.

Keep large archive processing benchmarks separate from one-FPO solver benchmarks. Scaling input records and scaling a mathematical planning problem are different workloads.

## 6. Hardware acceptance procedure

1. Identify delivered ESP32 and SHT30 probe; inspect signal labels against the current circuit. Do not rely on custom schematic pin numbering as board-header numbering.
2. Build and flash the chosen pinned firmware; retain compile/flash result and firmware version. Circuit-file generation is not firmware compilation.
3. Register the node and location assignment, configure device-scoped credentials and reachable broker, and test actual authentication/TLS.
4. Match a physical sample's device/boot/sequence identity, values, time-quality and firmware version to one backend record and its validation report.
5. Compare simulator and hardware contract behavior for the same valid/invalid payloads. Keep measured origin distinct from simulated origin.
6. Exercise power, Wi-Fi, broker and backend outages separately, including reboot before clock sync, duplicate replay and full buffer. Record observed retention limits and gaps.
7. Verify the intended on-screen condition display and planning-snapshot linkage. A real reading feeding a demo lot produces a demo proposal, not real FPO evidence.
8. Separately evaluate the approximation policy. Do not demonstrate dangerous grain deterioration or present a warm/humid sensor stimulus as proof of actual spoilage detection. A controlled environmental response shows measurement/pipeline responsiveness only.

One probe demonstrates one monitored point. Expansion to multiple bags/sheds requires a placement/coverage study and corresponding devices/assignments; no fixed bags-per-sensor claim is established.

## 7. Completion gates

- **Layer 1 accepted:** contracts/fixtures pass, an authorized real import is demonstrated for real mode, counts/coverage/manifests/replay are correct; unavailable real sources remain visibly blocked.
- **Layer 2 accepted:** rules, provenance, precision/mappings, readiness and corrections pass; empirical anomaly-performance claims require reviewed real evidence.
- **Layer 3 accepted for demo:** baseline/forecast fixture or evaluated model clearly identified; oracle/validator/integration checks pass under an explicit demo policy bundle. A model fixture never qualifies as evaluated forecasting.
- **Hardware accepted:** physical path passes bench and outage tests for recorded versions. Simulator-only success is insufficient.
- **Whole application accepted:** Layer 4–5 decisions frozen and approval → communication → actual sale → remainder → replan verified, with concurrency/failure cases.
- **Pilot/production readiness:** real policy/data/authority/deployment and field validation gates closed. Passing the college demonstration does not silently close them.

The future agent's completion report must name which gates actually passed, attach evidence, and leave all others NOT_RUN/BLOCKED/FAIL as appropriate.
