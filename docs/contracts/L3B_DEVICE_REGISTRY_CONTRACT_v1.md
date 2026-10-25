# L3b device registry slice `devices/v1` — devices, assignments, tenant-scoped readings

26 October 2026 • **Status: FROZEN** (contract version `devices/v1`) on the owner's approval of H-4a (CHAT-40). Schemas and examples: `kisaniti_implementation/contracts/devices/v1/`. Builds on the L3B sensor contract ([L3b_SENSOR_STORAGE_CONTRACT_v1.md](L3b_SENSOR_STORAGE_CONTRACT_v1.md) §3 and §4) and the access slice ([L0_ACCESS_CONTRACT_v1.md](L0_ACCESS_CONTRACT_v1.md)).

## 1. Rules

1. A **device** belongs to exactly one FPO, fixed when it is registered. It never moves: a device sold to another FPO is registered again under a new `device_id`. `device_id` is globally unique (the same string is the MQTT identity).
2. Registering a device is an administrator action (`users.manage` for now; a dedicated capability would be a new access slice version). `source_origin` (`DEVICE`, `WOKWI_TWIN`, `SIMULATOR`) is set at registration and the backend compares it with what the device claims; readings of simulators and twins are stored SYNTHETIC.
3. The **assignment history** (placement `IN_GRAIN` or `AMBIENT`, crop, `assigned_at`, `sample_interval_s`) is append-only. Entering a new assignment never edits an old one. The FPO of an assignment is the device's FPO, never taken from the request.
4. **FPO of a reading** (IC-DOMAIN-004): decided by the server, not by the device payload: the device's FPO from the registry; the placement valid at `measured_at` is the latest assignment with `assigned_at <= measured_at`. A reading whose time is not trusted (`measured_at_trusted` false) is stored and shown to the device's FPO but is **never matched to a placement by guessing**.
5. A reading of an unregistered device is quarantined and visible to no FPO.
6. Tenants see only their own devices and readings: another FPO's device answers **404 `NOT_FOUND`** exactly like a missing one. Enforced by row-level security on `devices`, `device_assignments` and `sensor_readings` (through the device), and again by explicit filters.
7. A tenant sees the `DeviceReadingV1` view of a reading, never the raw payload, its hash or the broker topic.
8. Broker credentials are not part of this slice (H-4b); no secret appears in any object here.

## 2. Endpoints (under `/api/v1`)

| Method and path | Needs | Does |
|---|---|---|
| `GET /devices` | `data.read` | the FPO's devices with current assignment and last trusted reading time |
| `POST /devices` | `users.manage`, CSRF | register a device (`RegisterDeviceRequestV1`); 409 if the id exists anywhere |
| `GET /devices/{id}` | `data.read` | one device; 404 for another FPO's |
| `POST /devices/{id}/assignments` | `data.write`, CSRF | append an assignment (`AssignmentRequestV1`); `assigned_at` defaults to now and may not lie in the future by more than 5 minutes |
| `GET /devices/{id}/assignments` | `data.read` | the full history, newest first |
| `GET /devices/{id}/readings?limit=` | `data.read` | the latest readings (default 100, at most 1,000) as `DeviceReadingV1` |
| `POST /devices/{id}/deactivate` | `users.manage`, CSRF | the device is no longer accepted by the ingestion service; history is kept |

## 2a. Amendment A1 — stored storage assessments (26 October 2026, before any consumer; CHAT-42)

The slice gains the stored result of the frozen rule `STORAGE-DEMO-0.1` (object `StorageAssessmentV1` of `contracts/sensor/v1`, unchanged):

| Method and path | Needs | Does |
|---|---|---|
| `GET /devices/{id}/assessment` | `data.read` | the latest stored assessment of the FPO's device; 404 `NOT_FOUND` with "no assessment yet" for its own device before the first run, "device not found" otherwise |
| `GET /devices/{id}/assessments?limit=` | `data.read` | the latest assessments, newest first (default 48, at most 1,000) |

Semantics of the **assessment job** (a system process, not a tenant request):

1. **Slots.** A device is assessed at whole-hour instants (UTC). A slot already assessed is never recomputed (idempotent, one row per device and slot).
2. **Catch-up.** After a gap, every missed slot is assessed in order, so the persisted state equals the state of an uninterrupted hourly run (the rule's reference behaviour, `assess_series`). A first run starts at the first whole hour after the current assignment began, but never more than 14 days before now; with no assignment at all, a single `NO_ASSIGNMENT` assessment is stored for the latest slot.
3. **Persisted hysteresis.** Every stored assessment carries the hysteresis state it ended with (`level`, `below_since`, and the assignment it belongs to). The next slot starts from the previous stored state; a new assignment resets it, as the rule requires.
4. **Placement at the time of the reading** (contract L3B §3, IC-DOMAIN-004). The assignment used at a slot is the latest with `assigned_at <=` the slot. A reading is used only if the assignment valid at *its own* `measured_at` is that same assignment. A reading with an untrusted or missing time is never matched, so it is stored and shown but never used.
5. **Append-only**, visible to a tenant only through its own devices (row-level security), written only by the job.
6. **Origin.** The result's origin follows the rule's taint: any simulator, twin or synthetic input makes it SYNTHETIC.

## 3. Ingestion

`kn-ingest` reads its registry from the `devices` table when started with `KN_REGISTRY=db` (PostgreSQL store). Changes made in the website take effect within the registry's refresh interval (30 seconds) or sooner for an unknown device (at most one lookup per 5 seconds). `devices.json` stays for stand-alone and SQLite use.

## 4. Not in this slice

Broker user creation and password issue (H-4b), several FPOs sharing a device, moving a device, calibration records, per-lot linking (comes with the lot slice), and any page. (The assessment job is amendment A1.)
