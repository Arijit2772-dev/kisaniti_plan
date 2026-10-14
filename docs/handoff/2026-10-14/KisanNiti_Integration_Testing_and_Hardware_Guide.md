# KisanNiti — Integration, Testing and Hardware Guide v0.2

> **14 October 2026 integration update:** Start with [Integrated Layer 1–3 handoff](KisanNiti_Layers_1_to_3_Handoff.md) and its acceptance plan. They consolidate this document and explicitly supersede obsolete hardware/location assumptions. Current hardware: ESP32-WROOM-32 + protected I2C SHT30 probe, with no manual moisture-meter requirement. Older hardware proposals below are retained only as history. No implementation/test completion is implied.

Date: 14 October 2026
Status: User-required testing commitment and proposed implementation/assembly plan. No application, firmware, physical wiring or integration tests have been executed by this documentation work.
User hardware status: nothing bought yet. Start with one physical sensor node. Layer 1's 20-device profile remains a simulated load/test profile, not a procurement requirement.
Applies across Layers 1–5 and their boundaries. The detailed Layer 4/5 policies still need design review; their tests will follow those agreed rules.

Recommendation revision: the user prioritizes easy assembly and assistant-assisted software maintenance, and has bought no hardware. Replace the first-purchase recommendation of a bare DHT22/DevKitC circuit with an ESP32 Feather V2 plus connector-equipped SHT40 breakout, using ESPHome for initial connected bring-up. This is a proposed hardware/firmware refinement, not a claim of completed compatibility testing or a validated warehouse deployment. Layer 1's earlier DHT22 selection is explicitly being reconsidered; sensor type, firmware and capabilities remain recorded in device metadata.

## 0. Evaluation and current recommendation
| Approach | Physical work | Software work | Decision |
| --- | --- | --- | --- |
| Bare ESP32 + DHT22 | breadboard, several wires, pull-up and pinout checks | custom firmware/driver integration | viable budget fallback; more manual assembly |
| ESP32 Feather V2 + SHT40 STEMMA QT breakout | plug matching sensor cable and USB | existing ESPHome driver, MQTT and OTA plus KisanNiti integration | recommended for first easy-to-assemble node |
| Proprietary cloud thermometer | often easy to power | API availability, accounts and data access depend on vendor | not selected for our own controllable integration |
| Fully custom PCB/industrial field device | engineering, manufacture and enclosure work | broader testing and maintenance | only after prototype and site requirements are established |

Selection basis is simple repeatable assembly, documented components, editable firmware, USB recovery, network updates and a provider-independent sensor contract. It is not a universal claim of lowest purchase price or best industrial reliability. Evaluate Indian availability, delivered cost and exact SKU before ordering; no local retailer/price has been verified in this review.
Also evaluated M5Stack ATOM Lite plus ENV-IV. The manufacturer's store marks ENV-IV end-of-life, so it is not our selected new-purchase reference. This does not mean already-owned working units are unusable.

## 1. Definition of complete
Every implemented layer must pass its meaningful component tests, contract tests, integration tests with its adjacent components, and applicable failure/recovery checks. The complete workflow must also pass end-to-end tests. A screenshot or a successful mocked test is not proof of real-provider or hardware readiness.
Track separately: software simulation passed; real-device bench passed; live external-provider integration passed; full workflow passed; field/pilot validation passed. Untested and blocked are explicit statuses.
Testing provides measured evidence for defined scenarios; it cannot establish zero possible bugs or scientific crop-condition validity without appropriate evidence.

## 2. Layer-by-layer evidence
| Area | Required checks | Evidence |
| --- | --- | --- |
| Layer 1 collection | source schema/access, counts, checkpoints, corrections, duplicates, source timestamps, real/synthetic separation | versioned source fixtures and authorized bounded live imports |
| Layer 2 cleaning/validation | unit conversion, mappings, missingness, flags, original retention, no silent deletion of genuine spikes, tenant isolation | explicit input/expected-output examples and real-source review audit |
| Layer 3 forecasting | chronological evaluation, data leakage prevention, baseline comparisons, declared forecast horizon and coverage | dataset/model manifests and held-out evaluation reports |
| Layer 3 optimizer | constraints, exact stock/financial arithmetic, full priority sequence, fairness across multiple lots, remainder, timeout semantics | tiny exhaustive oracles, independent PlanValidator, realistic benchmarks |
| Layer 4 decisions | approved permissions, immutable revisions, manager edits, freshness policy, simultaneous approval and reservation conflicts | real database transaction/concurrency tests after policy decisions |
| Layer 5 outcomes | communication failures/statuses, retry without false success, partial/multiple sales, duplicate sale events, reservation reconciliation | sandbox communication receipts, authoritative ledger checks and agreed-policy examples |
| Hardware path | measurement, firmware, device identity, transport, ingestion, historical assignment, dashboard, outages/replay | captured device logs and backend records with matching event IDs |

## 3. Shared interfaces and reproducibility
Before parallel implementation, freeze versioned schemas and examples for SensorReading, MarketObservation, lot/inventory events, PriceForecast, transport estimates, OptimizationInput/Result, plan revisions and approved communication requests.
Use canonical IDs, explicit units, timezone rules, event IDs and error codes. Consumer/producer contract tests must agree on valid, missing, invalid and duplicate cases. Firmware and simulator share the same reading contract.
Version code, firmware, policies, source mappings, model artifacts, datasets and test fixtures. Every result identifies the versions tested. API/database migrations require compatibility checks and a recovery path.
Use dependency stubs for fast deterministic tests, then separately test actual PostgreSQL, MQTT broker and provider adapters in a controlled environment. Test secrets and fictional contacts stay outside public logs and Git. External messaging uses an authorized test destination/channel.
Automate appropriate tests on implementation changes. Slow data/model evaluations and physical-device tests can run as deliberate suites; record when they were last run and which version they covered. Manual hardware steps must have expected observations and recorded results.

## 4. Complete application scenario
Create an authorized fixture FPO/member/lot and zone assignment; collect sensor and market data; validate; obtain a usable forecast and applicable route/quote; generate a plan; independently validate it; approve/reserve with current stock checks; record a test communication outcome; record a partial sale; verify remaining reservations and availability; create the next planning cycle.
Verify what should NOT change as well: a sensor message must not trigger model training or alter an approved plan; a proposal must not reserve stock; a duplicate sale must not deduct stock again.
Repeat with missing forecasts, provider outages, late readings, invalid credentials, interrupted imports, repeated messages, concurrent approvals, incomplete optimization, failed communication and off-plan sales under the policy eventually agreed. No unapproved reconciliation behavior is invented by a test fixture.

## 5. First physical purchase: one bench node
Reference board: Adafruit ESP32 Feather V2 with STEMMA QT connector; the pre-soldered-header version is product 5900. Reference sensor: Adafruit SHT40 STEMMA QT/Qwiic breakout, product 4885. This purchase is a breakout board, not the bare SHT40 chip.

| Item | Quantity | Requirement |
| --- | --- | --- |
| ESP32 Feather V2 with STEMMA QT | 1 | exact board/revision, product 5900 reference |
| SHT40 STEMMA QT/Qwiic breakout | 1 | product 4885 reference; connector already on board |
| STEMMA QT/Qwiic JST-SH four-pin cable | 1 | matching connectors at both ends, 100 mm reference product 4210 |
| USB data cable | 1 | USB-C at Feather end and a compatible computer end; not charge-only |
| Multimeter | optional/borrowed | troubleshooting if a hardware fault is suspected |
| Vented enclosure and stable USB supply | later | after bench checks; enclosure must expose sensor to representative air |

With those exact connector-equipped parts, no soldering, breadboard or external pull-up resistor is needed for the sensor connection. Do not substitute an arbitrary four-wire connector: Grove/HY2.0 and STEMMA QT/JST-SH are not interchangeable merely because both have four wires.
No Raspberry Pi, display, battery system, cellular modem or custom PCB is required for the first bench test. Computer and board need a usable network; the reference ESP32 uses 2.4 GHz Wi-Fi. Confirm signal at the intended storage location before scaling up.
The manufacturer's board/sensor pages were inspected for features; no current Indian price, local stock or delivered total is claimed. Do not buy both the previous bare-DHT22 kit and this connector kit. If local availability or price prevents this exact choice, compare a documented equivalent before substituting it.

## 6. Reference wiring
With USB power disconnected, plug the matching JST-SH cable into the Feather's STEMMA QT connector and into either matching sensor connector. Connect USB-C to the computer after inspecting the assembly. No loose signal-wire assignments are needed for this reference setup.
The Feather's battery connector is not the sensor connector. Use USB power for this prototype, no battery required. Do not apply the earlier DevKitC GPIO27/DHT22 wiring to the SHT40 I2C setup.
Firmware must configure the actual Feather V2 I2C pins and enable its switched STEMMA QT power supply (NEOPIXEL_I2C_POWER, GPIO2 driven high per board documentation). Do not assume an ESPHome generic board profile automatically performs this. Verify the selected board revision/pin map, scan for the sensor and inspect readings before accepting the configuration.
The Adafruit SHT40 breakout reference uses I2C address 0x44. A different module/address requires a checked adapter/configuration. Keep the sensor separated from board heat; the initial short cable is a bench arrangement, not an arbitrary warehouse cable-length specification.
Physical connection and device identity still need user observation; the assistant cannot certify an unseen assembly.

## 7. Hardware-to-application connection
SHT40 -> ESP32/ESPHome -> Wi-Fi -> authenticated MQTT broker -> ingestion worker -> Layer 2 checks -> PostgreSQL -> dashboard / supported StorageAssessment -> Layer 3 planning input.
MQTT is a lightweight message transport. The broker receives device messages and delivers them to the application subscriber. It is not the database and not an ML model.
Initial broker recommendation: Eclipse Mosquitto on the development computer or a reachable controlled server. The ESP32 connects to the computer/server's reachable address, not to localhost on the ESP32. A computer-only broker stops being available when that computer sleeps or disconnects.
Later use a stable hosted/private-network endpoint with TLS, authentication and topic access controls. Do not expose an anonymous development broker to the public internet.
Firmware recommendation for initial connected bring-up: ESPHome with its ESP32 and sht4x components, MQTT support and secured OTA. Pin compatible firmware/toolchain versions and test the actual board configuration. Home Assistant is not required for a standalone MQTT connection to our backend; disable unneeded discovery and avoid configuration expecting an absent API client.
ESPHome's existing sensor/network/update components reduce custom firmware work. KisanNiti still needs configured event payloads, identity, timestamps, validation, backend integration and failure handling; a default two-topic sensor example alone does not satisfy our full contract.
First flash is over USB. Subsequent supported updates can use authenticated/encrypted OTA over a reachable network; keep USB recovery and a known-good build. Validate OTA and credential recovery on the bench. Do not expose OTA directly to the public internet.
The earlier custom ESP-IDF path remains an option for advanced reliability features, not a second firmware project to build immediately.

## 8. Proposed device message and delivery rules
Capability distinction: basic ESPHome configuration does not establish our previously proposed power-loss-safe offline queue or database-commit acknowledgements. The connected prototype must report outages/gaps honestly. Persistent replay below remains an explicit unimplemented reliability target requiring a tested extension or custom ESP-IDF firmware before offline-recovery readiness is claimed. Do not silently remove the requirement or call the initial prototype production-complete.
Message fields: schema version, registered device ID, boot/session ID, sequence/event ID, measurement timestamp when trustworthy, uptime/time quality, temperature_C, humidity_percent, read status and firmware version. Backend adds receipt time and resolves authorized FPO/zone using assignment history.
Do not trust a device-supplied FPO/zone as authority to write into an arbitrary organization's records. A device reassignment does not relabel old observations.
Suggested topic family: kisaniti/v1/devices/{device_id}/telemetry, plus restricted acknowledgement/status topics. Devices can publish only their own telemetry/status and subscribe only to their allowed acknowledgement/control topics.
Use per-device credentials, server certificate/hostname verification for TLS, and protected local provisioning. No Wi-Fi passwords or device secrets in public source code/logs.
Proposed delivery: MQTT QoS 1, unique event identities, backend deduplication and bounded retries. QoS 1 may redeliver messages. A broker publish acknowledgement alone does not establish that PostgreSQL stored the reading.
For a recoverable path, keep a bounded device-side persistent queue and remove a reading only after an authenticated application acknowledgement issued after durable acceptance, including acceptance of a quarantined invalid reading with its status. Duplicate accepted events receive a safe repeated acknowledgement. Define permanent-reject handling explicitly rather than looping forever.
Proposed initial queue target: 288 measurement events, approximately 24 hours at the selected five-minute interval. This is a finite development target to test for storage capacity, flash wear, power-cut recovery and overflow. Report dropped/gap events explicitly; do not promise unlimited offline storage or zero loss.
If power or clock state is lost, preserve uncertainty. Use time synchronization and uptime/session context where reliable; do not invent measurement timestamps or label old uploads as current readings.
MQTT status/last-will helps report connectivity, but storage freshness uses measured evidence. Retained status must include interpretable timestamps/session context; retained old telemetry cannot silently appear fresh.

## 9. User and assistant responsibilities
User: obtain the four identified items; plug the sensor cable while unpowered; connect USB and make the Wi-Fi network available; approve any necessary OS device/driver interaction; press a physical reset/boot button if needed; perform requested reference/placement checks; mount the accepted device.
Assistant: prepare editable ESPHome configuration, compilation/upload workflow, broker setup, ingestion/software contracts, simulator, backend/dashboard code and automated test suites as implementation decisions are frozen. During active work, if the tools can access the connected USB port or reachable OTA endpoint, build/flash/read logs through the user's computer; if access is unavailable, provide the exact required local step. No future work is claimed complete by this plan.
Assistant access is through available tools on the computer/network during work, not an independent permanent connection to the physical sensor. Keep USB connected during development for diagnosis/recovery. A deployed collector can run independently of the chat once hosted; a laptop-hosted collector depends on that laptop staying awake and connected. Unattended assistant monitoring is not configured by this document.
Source code written successfully is not the same as a successful compile, flash, sensor measurement or end-to-end bench test. Each needs evidence.

## 10. Bench-to-integrated sequence
1. Identify the exact Feather V2 and SHT40 breakout and inspect the unpowered cable connection.
2. Confirm USB connection and run a simple board/serial test.
3. Enable the board's QT power, verify the I2C sensor appears and read locally with a diagnostic configuration. Proposed bench interval: 10 seconds. Leave the SHT40 heater off initially; enablement later needs deliberate handling because heating affects measurement availability. Do not use false zeros on read errors.
4. Compare settled readings with a suitable reference thermometer/hygrometer side by side. Account for both instruments' accuracy and environmental settling; agreement is a plausibility check, not traceable calibration.
5. Connect to the authenticated broker and inspect the exact message and device identity.
6. Run the backend subscriber and verify matching event IDs/values/timestamps in storage.
7. Show the real reading on the correct FPO/zone dashboard; distinguish it from simulator data.
8. Restore normal five-minute sampling and the explicitly development-only sensor-age policy. Run a proposed 24-hour bench soak; report received, duplicate, invalid, delayed and missing events. Its duration is an initial test choice, not a production reliability guarantee.
9. Exercise Wi-Fi outage, broker/backend restart, duplicate replay, board reset, uncertain clock and sensor failure. Disconnect the sensor cable only while powered off. Verify old readings stay old and current assessment becomes UNKNOWN when evidence is insufficient. Mark persistent offline-recovery tests pending until that capability has actually been implemented; test secured OTA and USB recovery separately.
10. Place the accepted node in a representative zone and test signal and readings there. Only then scale physical devices according to actual storage coverage needs.

## 11. Storage placement and scientific scope
The device measures ambient air temperature and relative humidity. It does not directly measure grain moisture, pesticide contamination, produce grade or remaining shelf life.
Use a vented arrangement with sensor separated from the ESP32/regulator heat and away from direct sun, wet surfaces, condensation and unrepresentative drafts. Do not seal the sensing element in an airtight electronics box or bury it inside grain under this ambient-monitoring design.
One zone sensor does not prove uniform conditions throughout a warehouse. Assess gradients, crop/storage context, interference and Wi-Fi/power coverage before deciding physical node count. Long warehouse sensor runs need a separately tested electrical/interface design; do not extend the bench wires arbitrarily.
Until supported crop-specific assessment rules are available, show readings, device health, age and UNKNOWN storage assessment. Demonstrating an artificial HIGH threshold must be a clearly labelled synthetic policy test, not a claim about crop safety.

## 12. First acceptance milestone
One physical node produces a real measured reading, the authenticated pipeline stores it once, the correct dashboard shows its measurement time and source, and disconnection/replay behave as specified. This establishes the hardware/software path.
Full application readiness additionally requires source access, model/solver evidence, later-layer policy completion, actual database concurrency checks, end-to-end workflow tests and appropriate field validation.

## Sources
- Recommended board: https://www.adafruit.com/product/5900
- Feather V2 pins and QT power: https://learn.adafruit.com/adafruit-esp32-feather-v2/pinouts
- Recommended sensor breakout: https://www.adafruit.com/product/4885
- Matching cable: https://www.adafruit.com/product/4210
- Sensor manufacturer: https://sensirion.com/products/catalog/SHT40
- ESPHome SHT4x: https://esphome.io/components/sensor/sht4x/
- ESPHome MQTT: https://esphome.io/components/mqtt/
- ESPHome secured OTA: https://esphome.io/components/ota/esphome/
- Evaluated but not selected for new purchase, ENV-IV marked EOL: https://shop.m5stack.com/products/env-iv-unit-with-temperature-humidity-air-pressure-sensor-sht40-bmp280
- The following DHT22/DevKitC sources document the earlier budget alternative, not the current first-purchase recommendation:
- Espressif reference board/pins/power: https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32/esp32-devkitc/user_guide.html
- DHT22 electrical connection and sample interval documentation: https://www.adafruit.com/product/385
- DHT sensor connection guide: https://learn.adafruit.com/dht/connecting-to-a-dhtxx-sensor
- ESP-MQTT: https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/protocols/mqtt.html
- ESP-IDF official TLS MQTT example: https://github.com/espressif/esp-idf/tree/master/examples/protocols/mqtt
- Community DHT driver: https://components.espressif.com/components/esp-idf-lib/dht
- Mosquitto authentication/access control: https://mosquitto.org/documentation/authentication-methods/
- Mosquitto TLS: https://www.mosquitto.org/man/mosquitto-tls-7.html
- One physical node, 10-second bench mode, 288-event queue and 24-hour soak are project development proposals. IO27 applies only to the earlier DHT22 fallback. None of these proposals is a manufacturer guarantee or completed test.
