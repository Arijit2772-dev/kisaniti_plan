# KisanNiti — Layer 1–3 review record

14 October 2026. This records a documentation/architecture review, not application test execution.

## Reviewed basis

- Complete protected consolidated architecture, 2,674 lines, including its numerical, inventory, objective, validation, revision and approval rules.
- Repository instructions, blueprint status, decision register and 21-entry unknown register.
- Current chat Layer 1 working baseline, Layer 2 proposal, Layer 3 proposal and integration/hardware guide.
- Current ESP32 + SHT30 schematic/drawing text; the constraint catalogue's invariant/non-decision index. The untracked catalogue remains untouched.
- Relevant primary/reference pages: sensor supplier, Sensirion datasheet reference, IRRI equilibrium-moisture explanation (search-index excerpt; direct page retrieval failed during this review), NIST outlier guidance, forecasting evaluation reference, Chronos-2 model card and SCIP project.

Earlier market-access probes and circuit rendering checks were recorded in preceding work. They were not repeated as live market-import or electrical tests in this review.

## Corrections and gaps surfaced

| Finding | Resolution in this review |
|---|---|
| Older Layer 1/hardware guide still recommended DHT22 or Feather/SHT40 | New integrated handoff names current ESP32-WROOM-32 + protected SHT30. Prior documents now carry a prominent supersession notice. |
| Manual moisture-meter kit appeared earlier | Explicitly excluded from current automatic monitoring requirement. |
| Collection-centre reference workflow conflicted with user farm-shed context | Separate farm-shed integration profile from retained synthetic collection-centre load profile; no universal storage/custody claim. |
| Twenty simulated devices could be mistaken for a purchase requirement | One physical node initially; 20-device fixture is a separate software load profile. |
| Air humidity risk approximation could be mistaken for grain moisture/spoilage measurement | Measurement/assessment contracts separated; crop-risk policy remains an explicit open gate. |
| Layer numbers could be mistaken for implementation completion | Logical Layers 1–3 distinguished from repository implementation phases. |
| Models/solver named without selection evidence | Baselines/candidates and promotion/benchmark gates retained; no trained model or production solver winner claimed. |
| Tests discussed without evidence | 48 named acceptance scenarios created with NOT_RUN status, separated from documentation checks. |
| Mutable provider data could undermine replay | Required versioned evidence, immutable evaluation basis and change-job contracts specified. |
| “Any agent can start” could conceal unfrozen exact schemas | Shared semantic contracts and ordered work packages defined; W01 explicitly freezes executable schemas before dependent coding. |

## Deliverables

- [Integrated handoff](KisanNiti_Layers_1_to_3_Handoff.md): 20 decision entries, 15 contract boundaries, 12 open gates and 13 work packages.
- [Acceptance plan](KisanNiti_Layers_1_to_3_Acceptance.md): 48 scenarios, small Layer 1→2→3 scenario, exact linked-sale example, device checks and completion gates.
- Four previous documents updated with a link to this consolidation. Historical text retained; no repository product code or protected architecture changed.

## Verification performed here

The accompanying [documentation-check record](KisanNiti_Layers_1_to_3_Document_Checks.json) records local checks of:

1. Unique, complete decision/contract/gate/work-package/test identifiers.
2. Every named shared contract is covered by at least one acceptance scenario; all referenced contract IDs exist.
3. Relative local links in the handoff, acceptance plan, review record and four updated prior documents resolve.
4. Fixture arithmetic, unit-conversion illustration, transport-return illustration, fairness fraction illustration and linked-sale inventory table agree with exact arithmetic.
5. Current drawing contains SHT30 and the specified four signal names.
6. Prior documents contain the current-handoff supersession notice.

These checks do not execute backend services, solver/model libraries, PostgreSQL, MQTT or firmware. Contract coverage means the tests are specified, not that tests exist in code or pass.

## Current evidence status

| Area | Status |
|---|---|
| Integrated design reconciliation and documentation checks | Completed; detailed check result in companion JSON |
| Executable API/schema/database contract freeze | Not complete |
| Accepted application implementation | Not started under current repository status |
| Authorized historical-data import and actual coverage | Not demonstrated by this review |
| Forecast training/evaluation and selected model | Not complete |
| Optimizer implementation/oracle/benchmarks | Not complete |
| Application component/integration/end-to-end tests | NOT_RUN |
| Circuit SVG/PNG | Existing deliverable; prior visual review recorded |
| KiCad electrical-rule check / PCB layout | Not performed; functional module symbols, no manufacturing footprints |
| Firmware compile/physical device/backend tests | No successful accepted build or real-device test established |
| Storage-risk science / field calibration | Open |
| Production/pilot readiness | Not established |

Unrequested unfinished prototype code under the workspace's `work/` area is excluded from the accepted design and deliverables. Its arbitrary demo thresholds, simplified optimizer or failed toolchain setup must not be mistaken for approved implementation or successful validation.

The next agent should continue from work package W00/W01 and the repository's current authority rules, not restart architecture or assume these documents lift its product-code gate. After Layers 4–5 are designed, extend the same contracts/test evidence into the complete workflow.
