# Hardware build specification — what to build, and how it connects

26 October 2026 • You build the node; I integrate it. This is the single sheet to follow. It collects what the repository already specifies ([HARDWARE_CURRENT.md](HARDWARE_CURRENT.md), `kisaniti_implementation/firmware/BUILD_GUIDE.md`, contract L3B v1) and adds the integration steps. Nothing here is a tested-assembly or availability guarantee: no real node has been built yet. Items I could not verify are marked **UNVERIFIED**.

## 1. What a node is

One ESP32 reads one protected SHT30 temperature and humidity probe pushed into the grain, every 5 minutes, and sends each reading over Wi-Fi to the server using MQTT with TLS. It keeps up to 288 readings (24 hours) through outages and resends until the server acknowledges. It measures one point; it does not measure grain moisture percentage, spoilage or shelf life.

## 2. Parts for one node (from HARDWARE_CURRENT.md)

| Part | Qty | Notes |
|---|---|---|
| ESP32-WROOM-32 development board, 38-pin, USB-C, headers soldered | 1 | built-in antenna; 2.4 GHz Wi-Fi only |
| Protected SHT30 I2C probe (the design reference is a PROBOTS PRO6111, about 50 cm cable) | 1 | probe wire colours differ by seller |
| 4.7 kΩ resistors | 2 | one pull-up on SDA, one on SCL |
| Breadboards (830/840 points) | 2 | |
| Jumper wires, male-to-male and male-to-female | 1 pack each | |
| 4-position 2.54 mm screw terminal | 1 | to bring the probe wires in |
| USB-C **data** cable | 1 | a charge-only cable cannot flash the board |
| 5 V / 2 A USB power adapter | 1 | |
| Box or base, cable ties, labels | 1 set | electronics stay outside the grain |
| A labelled, sealable container and about 1 kg of dried paddy | 1 set | not for food afterwards: the probe housing is not certified food-safe |

**Design conflict I found, to settle with you:** BUILD_GUIDE H7 and tracker F.6 still describe a one-time grain-moisture-meter check, while HARDWARE_CURRENT says no moisture meter is needed. The check would link the humidity bands to the moisture percentage farmers and buyers use, so I recommend keeping it as an optional extra, not as a purchase.

## 3. Wiring (do this with the USB cable unplugged)

| Probe wire | ESP32 | Note |
|---|---|---|
| VCC | 3V3 | never 5V or VIN |
| GND | GND | |
| SDA | GPIO 21 | 4.7 kΩ from SDA to 3V3 |
| SCL | GPIO 22 | 4.7 kΩ from SCL to 3V3 |

Read the probe's wire labels first, write down which colour is which, and photograph the label and the finished wiring. Use the pin names printed on the board. The firmware expects the sensor at address 0x44; some boards use 0x45, and in that case tell me before changing anything. Only the probe goes in the grain, at mid-depth.

## 4. What I give you, and what you give back

| Step | You | I provide | Check |
|---|---|---|---|
| 1. Bench | build, wire, photograph | firmware (`pio run -e wokwi -t upload` runs the real board without Wi-Fi) | serial shows `SHT30 found at 0x44` and a reading every 10 s; breathing on the probe raises humidity |
| 2. Identity | choose a device ID `kn-node-001` (letters, digits, hyphens, 3–32 characters, as the schema requires) | a broker user and password for that ID, the server address, the server's CA certificate | I register the ID with source `DEVICE` before it connects |
| 3. Configuration | copy `secrets.example.h` to `secrets.h`; fill in Wi-Fi name and password, server host, port 8883, device ID, device password, CA certificate text | a provisioning script (TO BUILD) that writes `secrets.h` from the registry, so no value is typed by hand | `secrets.h` is never committed or sent in chat |
| 4. Flash and first connection | `pio run -e esp32dev -t upload`, `pio device monitor` | the server running | serial shows `mqtt connected`, `TX …`, `ack seq=…` |
| 5. Acceptance | leave it publishing for 10 minutes | `kn-conformance publisher --device-id kn-node-001 --origin DEVICE` and the report | no FAIL; warnings explained |
| 6. Calibration and tests | salt check, settling time, outage tests, one-week run as in BUILD_GUIDE H4–H6 | `kn-report`, `kn-assess`, a write-up that does not overstate | records saved in `firmware/records/` |
| 7. Live | place the node, assign it to a lot in the website | device registry and assignment (H-4a), scheduled assessment (H-5), live page (H-6) | the storage page shows the node with no code change |

Send back: the photos, the serial log of step 4, the conformance output, and the filled calibration log. Never send passwords or the contents of `secrets.h`.

## 5. How it connects (so the build matches the integration)

```text
SHT30 ──I2C──► ESP32 ──Wi-Fi 2.4 GHz──► internet ──TLS :8883──► Mosquitto ──► kn-ingest ──► PostgreSQL ──► API ──► website
                     ◄─────────── ack (same boot_id, seq) ◄───────────┘
```

- Topics: telemetry `kisaniti/v1/devices/{device_id}/telemetry`, ack `…/ack`, status `…/status`. The device's broker user can use only its own topics.
- Message: `DeviceTelemetryV1` JSON, at most 1 KiB, with `device_id`, `boot_id`, `seq`, `measured_at`, readings and `source_origin`. A reading counts once per `(device, boot, seq)`; the device deletes it only after the ack.
- Which FPO and lot a node belongs to is decided on the server from the assignment history, never by the device.
- A server certificate from a public authority needs the matching root certificate in `secrets.h` instead of our test CA. **UNVERIFIED** which root the ESP32's TLS stack needs for the current chain; the conformance run against the public broker will show it before you rely on it.

## 6. Limits and what I need to know before you buy or build

- **Wi-Fi is required** (2.4 GHz). If the real farm has no Wi-Fi, tell me before building: a cellular variant is a new firmware profile and a new design decision, not a setting.
- **Power:** the design assumes a USB adapter. Battery or solar operation is not designed.
- **Buffer:** readings not yet sent are lost on a power cut (RAM buffer). A flash-backed queue is planned (H-7).
- **Accuracy:** one probe, ±2 %RH typical in the mid range (datasheet), so a salt-calibration offset is measured before any result is used. Thresholds are desk-research values until the bench tests are done.
- **Not covered:** more than one node per lot, other sensor types, weighbridge or gate hardware. Adding them means a new contract version and the same conformance test (H-9); do not change pins or messages by hand.
- Seller names, prices and delivery times are not verified here.

## 7. What I will build so that your part is only assembly

1. Provisioning script that produces `secrets.h` and registers the device (HW-P1).
2. Device registry, assignment and per-device broker credentials (H-4a, needs the login work first).
3. Storage assessment job and the live device page in both languages (H-5, H-6).
4. Hardware acceptance report generator that turns the serial log, conformance output and calibration log into one record (HW-P2).
5. A node self-check page in the serial output that prints the same checklist as §4 step 1.
