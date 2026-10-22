# L3b contract slice v1 — device telemetry, SensorReading and StorageAssessment

**Status: FROZEN for implementation (W01 slice for Layer 3b), 22 October 2026.** Recorded in the [decision register](../blueprint/01_DECISION_REGISTER.md) as `W01-L3B`. Machine-readable JSON Schemas: `kisaniti_implementation/contracts/sensor/v1/` (they reuse `shared/v1/common.schema.json`).

**Aligned with the [shared contract](CONTRACT_SPEC_V1_shared.md) (22 Oct, same day):** every time below is a shared-contract *instant* (`YYYY-MM-DDTHH:MM:SS.sssZ`, exactly milliseconds), and every stored reading and assessment carries the shared `provenance` object.

This slice turns the existing design text into exact fields. It does not add new policy. Sources:

- C04 `SensorReadingV1` and C09 `StorageAssessmentV1` in the [Layers 1–3 handoff](../handoff/2026-10-14/KisanNiti_Layers_1_to_3_Handoff.md) §3 and §10;
- message, topic and delivery rules in the [integration and hardware guide](../handoff/2026-10-14/KisanNiti_Integration_Testing_and_Hardware_Guide.md) §8;
- the storage rule `STORAGE-DEMO-0.1` in [Patiala research Part B](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md);
- hardware CHAT-29 (ESP32-WROOM-32 + SHT30 probe in the grain).

Changing any field, enum or rule below needs a new version (`…v2`) and a register entry. Adding an optional field is allowed as a minor change if old consumers can ignore it.

## 1. Flow

```text
SHT30 probe ─I²C─► ESP32 firmware ─MQTT (TLS in real use)─► ingestion adapter
   DeviceTelemetryV1  ──validate, de-duplicate, store──►  SensorReadingV1
   SensorReadingV1 (24 h window) + DeviceAssignment ──core.storage──► StorageAssessmentV1
```

## 2. DeviceTelemetryV1 (what the device publishes)

Topic: `kisaniti/v1/devices/{device_id}/telemetry`, QoS 1, not retained. JSON, UTF-8, at most 1 KiB.

| Field | Type | Rule |
|---|---|---|
| `schema` | const `"kisaniti.device_telemetry.v1"` | |
| `device_id` | string `^[a-z0-9][a-z0-9-]{2,31}$` | Must equal the `{device_id}` in the topic. Identity only; it never grants FPO or location authority |
| `boot_id` | string, 8 lowercase hex | New random value at every boot |
| `seq` | integer ≥ 0 | Starts at 0 each boot, +1 per sample (not per send) |
| `measured_at` | instant or `null` | `null` when the clock was never trustworthy for this sample |
| `time_quality` | `NTP_SYNCED` \| `UPTIME_BACKFILLED` \| `UNSYNCED` | `UPTIME_BACKFILLED`: sample taken before NTP sync, time filled in later in the same boot from the uptime difference |
| `uptime_ms` | integer ≥ 0 | Milliseconds since boot at measurement |
| `temperature_c` | number −40…125 or `null` | `null` when the read failed. Never a false zero |
| `humidity_pct` | number 0…100 or `null` | As above |
| `read_status` | `OK` \| `CRC_ERROR` \| `NO_ACK` \| `TIMEOUT` | Sensor read outcome. Values are `null` unless `OK` |
| `sensor_model` | string, e.g. `"SHT30"` | |
| `firmware_version` | string | e.g. `"kn-fw-1.0.0"` |
| `source_origin` | `DEVICE` \| `WOKWI_TWIN` \| `SIMULATOR` | Set at build time. Twin and simulator data are never shown as real |
| `sample_interval_s` | integer > 0 | Configured interval for this boot |
| `resent` | boolean | `true` when this sample is sent again from the buffer |
| `dropped_since_boot` | integer ≥ 0 | Samples lost to buffer overflow in this boot |

**Event identity:** `(device_id, boot_id, seq)`. A repeated identity is a duplicate. If the payload of a duplicate differs from the stored one, the backend keeps the first and records a conflict.

**Acknowledgement:** after the backend has stored (or quarantined) a reading it publishes `{"schema":"kisaniti.device_ack.v1","boot_id":…,"seq":…,"status":"ACCEPTED"|"DUPLICATE"|"QUARANTINED"}` on `kisaniti/v1/devices/{device_id}/ack` (QoS 1). The device deletes a buffered sample only after this ack (mode `APP_ACK`), or after the broker's QoS 1 PUBACK when no backend acknowledges (mode `BROKER_ACK`, used only for the stand-alone Wokwi twin). A broker PUBACK is not proof that the database stored the reading.

**Buffer (firmware v1):** RAM ring buffer of 288 samples (24 h at 5 min). It survives Wi-Fi, broker and backend outages. **It does not survive a power cut or reset**; samples in it are then lost and the gap shows as missing sequence numbers in the old boot. On overflow the oldest sample is dropped and `dropped_since_boot` increases. A flash-backed queue is future work.

**Status topic:** `kisaniti/v1/devices/{device_id}/status`, retained, `{"state":"online"|"offline","boot_id":…,"at":instant|null}`; `offline` is the MQTT last will. Status never makes a reading fresh.

## 3. SensorReadingV1 (stored by the backend)

Every field of `DeviceTelemetryV1`, plus:

| Field | Type | Rule |
|---|---|---|
| `reading_id` | string | `"{device_id}:{boot_id}:{seq}"` |
| `received_at` | instant | Backend clock at first receipt |
| `measured_at_trusted` | boolean | `true` only for `NTP_SYNCED` or `UPTIME_BACKFILLED`, and `measured_at` not more than 5 min after `received_at` |
| `temperature_valid`, `humidity_valid` | boolean | Value present and inside the plausible range (temperature −10…60 °C, humidity 0…100 %) |
| `ingest_status` | `ACCEPTED` \| `QUARANTINED` | Quarantined readings are kept (with reason) but never used for assessment |
| `quarantine_reason` | string or `null` | e.g. `TOPIC_DEVICE_MISMATCH`, `SCHEMA_INVALID`, `UNREGISTERED_DEVICE`, `FUTURE_TIMESTAMP` |
| `raw_sha256` | hex string | Hash of the exact payload bytes |
| `topic` | string | MQTT topic it arrived on |
| `provenance` | shared `provenance` | From the **registered** source, not the payload: `DEVICE` → `REAL`/`SENSOR`; `WOKWI_TWIN` or `SIMULATOR` → `SYNTHETIC`/`SIMULATOR` with a generator. A payload whose `source_origin` disagrees with the registration is quarantined (`SOURCE_ORIGIN_MISMATCH`) |

The backend, not the device, resolves the FPO, location and placement from the device-assignment history at `measured_at` (IC-DOMAIN-004). A reading with an untrusted time cannot be assigned by guessing; it stays stored but is not used for assessment.

## 4. DeviceAssignment (input to the rule)

| Field | Type | Rule |
|---|---|---|
| `device_id` | string | |
| `placement` | `IN_GRAIN` \| `AMBIENT` | `IN_GRAIN`: probe tip at mid-depth of the bag or heap |
| `crop` | `PADDY` \| `WHEAT` | `PADDY` includes basmati, the main stored crop of the study (CHAT-32). Same thresholds for both (Part B4) |
| `assigned_at` | instant | Insertion or reassignment time; starts the settling period |
| `sample_interval_s` | integer | 300 for real use |

## 5. StorageAssessmentV1 (output of `STORAGE-DEMO-0.1`)

| Field | Type | Rule |
|---|---|---|
| `schema` | const `"kisaniti.storage_assessment.v1"` | |
| `policy_version` | const `"STORAGE-DEMO-0.1"` | |
| `device_id`, `crop`, `placement` | | From the assignment |
| `assessed_at` | instant | |
| `window_start`, `window_end` | instant | 24 h ending at `assessed_at` |
| `category` | `LOW` \| `MEDIUM` \| `HIGH` \| `UNKNOWN` | |
| `unknown_reasons` | list of `NOT_IN_GRAIN`, `NO_ASSIGNMENT`, `SETTLING`, `STALE`, `LOW_COVERAGE` | Empty unless `UNKNOWN`; all failing checks are listed |
| `mould_level` | `LOW` \| `MEDIUM` \| `HIGH` or `null` | Water-activity band after hysteresis, before temperature adjustment |
| `adjustments` | list of `COLD_BELOW_10C_LOWERED`, `WARM_25C_RAISED` | |
| `aw_median` | number or `null` | Median RH / 100 over the window |
| `rh_median_pct`, `temperature_median_c` | number or `null` | Reported also for `AMBIENT` placement, labelled "ambient conditions" |
| `insect_flag` | `INSECT_FAVOURABLE` \| `INSECTS_SLOW` \| `INSECTS_INACTIVE` or `null` | Advisory; never sets urgency alone |
| `samples_used`, `samples_expected` | integer | Coverage check: `samples_used ≥ 0.75 × samples_expected` |
| `latest_measured_at` | instant or `null` | |
| `hysteresis` | object `{level, below_since}` | State carried to the next assessment |
| `label` | const `"Experimental approximation — not measured spoilage"` | Shown wherever the category is shown |
| `evidence_ref` | string | Link to Part B |
| `provenance` | shared `provenance` | `source_kind = DERIVED`; `origin` by the taint rule over the readings used (twin data gives `SYNTHETIC`) |

### 5.1 Rule `STORAGE-DEMO-0.1` (exact)

Constants: `SETTLING = 6 h`, `WINDOW = 24 h`, `FRESH = 15 min`, `COVERAGE = 0.75`, `AW_MEDIUM = 0.65`, `AW_HIGH = 0.70`, `HYST_MARGIN = 0.02` (2 RH points), `HYST_HOLD = 48 h`, `T_COLD = 10 °C`, `T_WARM = 25 °C`, `AW_WARM = 0.60`, insect bands 15 °C and 25 °C.

1. **Usable readings:** `ACCEPTED`, `measured_at_trusted`, both channels valid, `measured_at` in `(assessed_at − 24 h, assessed_at]` and `≥ assigned_at + 6 h`.
2. **Pre-conditions** (each failure adds a reason; any reason gives `UNKNOWN`):
   - `NO_ASSIGNMENT` — no assignment at `assessed_at`;
   - `NOT_IN_GRAIN` — placement is `AMBIENT` (medians still reported);
   - `SETTLING` — `assessed_at < assigned_at + 6 h`;
   - `STALE` — no usable reading in the last 15 min;
   - `LOW_COVERAGE` — usable readings `< 0.75 × (24 h / sample_interval)` (216 of 288 at 5 min).
3. **Medians:** `aw = median(RH)/100`, `T = median(T)` over the usable readings.
4. **Raw mould band:** `aw < 0.65` LOW; `0.65 ≤ aw < 0.70` MEDIUM; `aw ≥ 0.70` HIGH.
5. **Hysteresis** on the mould band (state `level`, `below_since`):
   - no previous level → take the raw band;
   - raw band above `level` → escalate at once, clear `below_since`;
   - raw band below `level` → de-escalate only if `aw ≤ lower_bound(level) − 0.02`, continuously for 48 h (`below_since` marks the start); the new level is then the raw band; if the margin is not met, clear `below_since`;
   - an `UNKNOWN` assessment leaves the state unchanged; a new `assigned_at` resets it.
6. **Temperature adjustment** of the hysteretic level: `T < 10` → one level lower (LOW stays LOW); else if `T ≥ 25` and `aw ≥ 0.60` and level is LOW → MEDIUM.
7. **Insect flag:** `T ≥ 25` favourable; `15 ≤ T < 25` slow; `T < 15` inactive.

### 5.2 What it does not claim

It estimates conditions that favour mould and insects at one probe point. It does not detect spoilage, measure moisture %, predict shelf life or speak for other bags. Thresholds are desk-research values (FAO) and are not calibrated locally; calibration steps are in `kisaniti_implementation/firmware/BUILD_GUIDE.md`.

## 6. Failure states summary

| Situation | Device | Backend | Assessment |
|---|---|---|---|
| Sensor unplugged / CRC fail | sends `read_status` ≠ OK, values `null` | stores, channels invalid | may become `STALE` |
| Wi-Fi or broker down | buffers ≤ 288, then drops oldest | — | `STALE` after 15 min |
| Power cut / reset | buffer lost, new `boot_id` | gap visible as missing `seq` | `STALE`, then `LOW_COVERAGE` until refilled |
| No NTP yet | `measured_at` null, `UNSYNCED`; backfilled later in the same boot if sync arrives | untrusted until backfilled | reading not used |
| Duplicate delivery (QoS 1) | — | `DUPLICATE` ack, no second row | — |
| Probe in open air | — | — | `UNKNOWN` (`NOT_IN_GRAIN`), ambient medians shown |
