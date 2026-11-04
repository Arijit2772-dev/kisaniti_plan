# Storage node — final build and connection guide

2 November 2026 • The one sheet to follow to get the parts for, build, flash, connect and demonstrate the KisanNiti storage node. The parts list and packaging are decision CHAT-50. The work order across chats is in [BUILD_LEDGER.md](BUILD_LEDGER.md) §2b. It is based on the existing design ([HARDWARE_BUILD_SPEC.md](HARDWARE_BUILD_SPEC.md), [HARDWARE_CURRENT.md](HARDWARE_CURRENT.md), product `firmware/BUILD_GUIDE.md`, contract L3B v1) and was re-checked against the code on 31 Oct (section 8). The electronics, firmware and message contract did not change. One bug was fixed (BUG-009, broker certificate). Calibration, outage and week-run details stay in `firmware/BUILD_GUIDE.md` H4–H6.

## 1. What the node is and is not

One ESP32 reads one SHT30 probe pushed into a container of grain every 5 minutes. It sends each reading over 2.4 GHz Wi-Fi as MQTT over TLS (port 8883), keeps up to 288 readings (24 h) through an outage, and deletes a reading only after the server confirms it was stored. The server turns the readings into a storage category (LOW / MEDIUM / HIGH / UNKNOWN, rule STORAGE-DEMO-0.1), which the website shows next to the lot.

It measures air temperature and relative humidity at one point in the grain. It does **not** measure grain moisture %, detect spoilage or predict shelf life.

## 2. Parts list (CHAT-50; status: not yet received)

The college supplies items 1–17. The team brings items 18–23. **Items 1–3 must not be substituted without checking first:** a different chip or sensor may not work with the firmware, and the terminal board must fit the exact ESP32.

| # | Item | Qty | Source | Exact spec / why |
|---|---|---|---|---|
| 1 | ESP32-WROOM-32 board | 2 (1 spare) | College | 38-pin, USB-C, CP2102, pins already soldered. Firmware target `esp32dev`. Not S2, S3 or C3 |
| 2 | SHT30 probe | 1 | College | Probots SKU PRO6111: metal housing, 50 cm cable, I2C. Not SHT20, SHT40, AHT, DHT or BME, and not the bare GY-SHT30-D module |
| 3 | ESP32 38-pin screw-terminal expansion board | 1 | College | Must fit item 1: 38-pin boards come in different widths. **Fit not verified** |
| 4 | 4.7 kΩ resistor, ¼ W | 2 | College | I2C pull-ups |
| 5 | Breadboard, 830-point | 1 | College | First tests and backup only |
| 6 | Jumper wires, male-to-male | 1 pack | College | First tests |
| 7 | Jumper wires, male-to-female | 1 pack | College | First tests |
| 8 | USB-C data cable | 1 | College | Not charge-only |
| 9 | USB wall adapter, 5 V / 2 A | 1 | College | Mains power for the run |
| 10 | Plastic (ABS) junction box | 1 | College | Big enough for items 1 and 3 together; not metal, which blocks Wi-Fi |
| 11 | Cable gland, PG7 or the size that fits the probe cable | 2 | College | Probe cable and USB cable. A USB plug may not pass through a small gland: check against item 8 |
| 12 | Standoffs or double-sided foam tape | 1 set | College | Fix the board in the box |
| 13 | Cable ties | 1 small pack | College | |
| 14 | Labels and marker | 1 set | College | |
| 15 | Small screwdriver | 1 | College | Terminals |
| 16 | Wire stripper | 1 | College | Only if the probe wire ends are not bare |
| 17 | Multimeter | 1 | College | Finding wiring faults |
| 18 | Sealable plastic container, about 2 L | 1 | Team | Holds the test grain |
| 19 | Dried basmati **paddy** (in husk, not polished rice) | 1 kg | Team | Rule and paper are for paddy. Not for food afterwards |
| 20 | Non-iodised salt | 1 small packet | Team | Calibration |
| 21 | Small cup | 1 | Team | Calibration |
| 22 | Glass jar with lid, or zip bag | 1 | Team | Calibration |
| 23 | Spray bottle | 1 | Team | Humid-grain test (20 ml) |

Also needed, not bought: this Mac, and a phone hotspot or 2.4 GHz router with internet (campus Wi-Fi often needs a login page, so do not rely on it). Not needed: moisture meter (BUILD_GUIDE H7 stays an optional one-time check at a mandi), battery, SIM module, custom circuit board, soldering.

Keep all parts until the demo is finished: the unit must run for days beforehand. Listing prices seen on 2 Nov 2026, not guarantees: board about ₹423 + GST, probe about ₹1,999 with GST (Probots).

## 3. Hardware architecture and wiring (USB unplugged)

```text
 sealed grain container        closed ABS box (outside the grain)
┌─────────────────────┐       ┌───────────────────────────────┐         5 V USB
│ SHT30 probe (I2C)   │─gland─│ screw-terminal board + ESP32  │─gland─── wall adapter
│ at mid-depth        │ 50 cm │ 2 × 4.7 kΩ pull-ups to 3V3    │
└─────────────────────┘       └───────────────┬───────────────┘
                                              │ 2.4 GHz Wi-Fi (phone hotspot or router)
                                              ▼
 this Mac / later a server:  Mosquitto :8883 (TLS, per-device user) ──► kn-ingest ──► PostgreSQL
                             ▲ ack only after the reading is stored ◄──┘                │
                                                                       assessor (hourly)┤
                                                website: readings, trend, category ◄ API┘
```

| Probe wire (read the label) | Terminal / ESP32 pin (printed name) | Pull-up |
|---|---|---|
| VCC | **3V3** (never 5V or VIN) | — |
| GND | GND | — |
| SDA | **GPIO 21** (often printed `D21` or `21`) | 4.7 kΩ from SDA to 3V3 |
| SCL | **GPIO 22** (often printed `D22` or `22`) | 4.7 kΩ from SCL to 3V3 |

- **Order:** wire it first on the breadboard, pass 5.1, then move it to the terminal board and pass 5.1 again. Only then close the box.
- **Pull-ups on terminals:** each resistor shares a terminal with a probe wire and 3V3, which may be a tight fit. This is not yet tested with these parts; check it on the bench.
- Probe wire colours differ by seller. Send a photo of the label before first power-on.
- Some probes have pull-ups inside. The two extra resistors are still safe.
- The firmware expects I2C address 0x44. The PRO6111 listing does not confirm this. If it is 0x45, the firmware needs a change: report it, do not edit by hand.
- Only the probe goes in the grain.
- Save photos of the label and finished wiring as `firmware/records/H1_wiring.jpg`.

## 4. Laptop setup (one time)

- PlatformIO is already installed on this Mac: `~/.platformio/penv/bin/pio`.
- Docker Desktop is running with the KisanNiti stack (`docker compose up --build -d --wait` in `kisaniti_implementation`).
- Connect the board by USB and run `ls /dev/cu.*`. A new `cu.usbserial-…` or `cu.SLAB_USBtoUART` should appear. If nothing appears, the cable is charge-only, or the board's CH340 driver is missing.

## 5. Build, step by step (each step has a check; do not go on until it passes)

### 5.1 Bench check, no Wi-Fi needed

```bash
cd ~/Documents/kisaniti_implementation/firmware
~/.platformio/penv/bin/pio run -e wokwi -t upload
~/.platformio/penv/bin/pio device monitor -b 115200
```

**Pass:** `[kn] SHT30 found at 0x44`, then a line every 10 s like `[kn] sample seq=3 T=27.10C RH=55.20% …`. Breathe on the probe and RH should rise within a minute.
**Fail:** `read_status=NO_ACK` means wiring, VCC not on 3V3, or a missing pull-up. `CRC_ERROR` means a loose wire. If the upload hangs at "Connecting…", hold the board's **BOOT** button until the upload starts.

### 5.2 Network: the laptop and the node on one phone hotspot

- The node needs **2.4 GHz** Wi-Fi.
  - iPhone: Settings → Personal Hotspot → turn on **Maximize Compatibility**.
  - Android: set the hotspot band to 2.4 GHz.
- Join the laptop to the hotspot and note its address with `ipconfig getifaddr en0`, for example `172.20.10.2` (iPhone) or `192.168.43.x` (Android).
- If the hotspot later gives the laptop a different address, redo 5.3.
- If macOS asks whether Docker may accept incoming connections, answer Allow.

### 5.3 Broker certificate and passwords

```bash
cd ~/Documents/kisaniti_implementation/backend/ingestion/broker
rm -rf certs                               # only when the laptop's address changed; a node then needs reflashing with the new CA
./setup_stack.sh <laptop-ip>               # certificates if absent, the ingestion broker user, ingestion.password
./check_esp32_tls.sh <laptop-ip>           # must print PASS (BUG-009 check; needs Docker)
./add_user.sh kn-node-001                  # prompts for the node's broker password; keep it out of the repository and chat
chmod 644 passwd
```

**Pass:** `PASS: the ESP32 will accept this certificate for <laptop-ip>`. Certificates made before 4 Nov 2026 lack the name `broker`, and `setup_stack.sh` refuses them: regenerate. HW-S2 will replace the manual `add_user.sh` step.

### 5.4 Register the node on the website

Sign in at http://localhost:3000 as the demo FPO admin and go to **Storage sensors**:

1. Register device `kn-node-001` with source **DEVICE (real node)**.
2. Assign it: placement **In grain**, crop **Paddy**, interval **300 s**. The interval must match the firmware.
3. On **Lots**, link the sensor to the demo lot.

The server, not the device, decides which FPO and lot the readings belong to.

### 5.5 Start the broker and the ingestion worker (registry from the database)

```bash
cd ~/Documents/kisaniti_implementation
docker compose --profile hardware up --build -d --wait
docker compose --profile hardware kill -s HUP broker     # makes a running broker read users added since it started
docker compose --profile hardware logs ingest | tail     # must show: subscribing kisaniti/v1/devices/+/telemetry
scripts/hw_stack_check.sh                                # all PASS (software node over MQTT and TLS through to the website)
```

Stop with `docker compose --profile hardware down`; without the profile, the broker and worker keep running.

**Before flashing, rehearse with the software node.** The last command above does this: it creates its own fictional FPOs and simulated device and removes its broker user afterwards. **Pass:** `ALL PASS`. To see readings on the Storage sensors page, run `scripts/demo_storage.sh`.

### 5.6 Flash the real profile

```bash
cd ~/Documents/kisaniti_implementation/firmware
cp include/secrets.example.h include/secrets.h    # git-ignored; never commit or paste it anywhere
```

Fill in `secrets.h`:
- the hotspot name and password;
- `KN_MQTT_HOST` = the laptop IP, exactly as used in 5.3;
- `KN_MQTT_USER` and `KN_DEVICE_ID` = `kn-node-001`, plus its broker password;
- the text of `backend/ingestion/broker/certs/ca.crt`, one quoted line per PEM line ending in `\n`.

Then:

```bash
~/.platformio/penv/bin/pio run -e esp32dev -t upload
~/.platformio/penv/bin/pio device monitor -b 115200
```

**Pass**, in this order: `wifi connecting to …`, `clock synced …`, `mqtt connecting to <ip>:8883`, `mqtt connected`, then every 5 minutes `TX kisaniti/v1/devices/kn-node-001/telemetry …` followed by `ack seq=… buffered=0`. The worker logs `-> NEW`, and the Storage sensors page shows the reading marked as real.

**Fail:**
- No `clock synced`: the hotspot has no internet, so NTP cannot be reached. TLS waits for the clock.
- `mqtt connect failed` with an `lastError` repeating: certificate or host mismatch; redo 5.3 and check the IP.
- `returnCode=4` or `5`: wrong broker user or password.
- `TX` but no `ack`: the worker is not running, or the device is not registered as DEVICE.

### 5.7 Acceptance

While the node publishes, run `python -m kisaniti_ingestion.conformance publisher --device-id kn-node-001 --origin DEVICE --seconds 900 …`. Use its own broker user with the ingestion rights, as in `e2e/run_sim_conformance_e2e.sh`, and stop `kn-ingest` while it runs, because the harness plays the backend. **Pass:** `"fail": 0`. Save the serial log and this output in `firmware/records/H2_first_run.txt`.

Then do the salt calibration, settling time and outage tests (BUILD_GUIDE H4–H5). Keep the records; the paper uses only measured values.

## 6. Demo plan (what the audience sees)

The storage category needs history. Readings from the first **6 h** after placing the probe are ignored (settling), and the rule needs **75 % of a day's samples** (216 of 288). So a node placed in grain shows **UNKNOWN for about 24 hours**, then a real category. Live readings and the trend appear at once.

| When | Do |
|---|---|
| Demo day −2 | Build, pass 5.1–5.7 |
| Demo day −1, at least **30 h** before | Probe at mid-depth in the labelled grain, container sealed, node on the wall adapter, laptop awake on the hotspot with the broker and worker running |
| Demo day, 1 h before | Serial monitor shows `ack`; website shows readings from the last 5 minutes and a category that is not UNKNOWN |
| During the demo | (1) node and serial monitor; (2) website Storage sensors: live reading, trend, category; (3) breathe on or warm the probe and the next reading shows it; (4) unplug Wi-Fi for 10 minutes, reconnect, and the buffered readings arrive with their original times; (5) the lot shows its storage category; the plan → approval → WhatsApp → sale chain continues from there |
| Fallback if anything fails | `scripts/demo_storage.sh` (30 h of simulator readings through the real ingestion code and the website, labelled SYNTHETIC), the Wokwi twin, and `scripts/chain.py`. The real node is shown as "built", never as "working" unless it is |

The humid-grain demonstration (category rising to MEDIUM/HIGH) takes days in real grain (BUILD_GUIDE H6). Show it from the recorded week run, not live.

## 7. Limits to state honestly

- A power cut loses readings not yet sent (RAM buffer; a flash queue, H-7, is planned).
- The node needs Wi-Fi; there is no cellular variant.
- It runs on mains USB only; battery and solar are not designed.
- There is one probe per lot.
- The thresholds are desk-research values until the bench tests are done.
- The calibration offset is not yet applied by the rule; that needs a small contract change once a real offset is measured.

## 8. What was verified on 31 October 2026, and what was not

| Check | Result |
|---|---|
| Firmware host tests (`pio test -e native`) | 13 of 13 pass |
| Firmware builds, real profile and twin | both build (RAM 19.6 %, flash 71.6 % for the real profile) |
| Live path: Mosquitto (TLS + per-device ACL) → `kn-ingest` → PostgreSQL, registry from the database | 13 of 13 receiver contract cases pass; software node 48 of 48 readings acknowledged and stored, sequence complete; publisher conformance 0 fail; the owning FPO sees the readings and another FPO sees 0 (row-level security). Run three times: the first two printed FAIL only because the script counted rows left by earlier runs in the test database (48 of 48 per boot both times); the check now counts only its own boot and the third run printed PASS |
| Website path on the running stack (`scripts/demo_storage.sh`) | device registered and assigned through the website, 360 readings acknowledged, assessment job stored 30 hourly assessments, website returned category HIGH from 277 of 288 samples, labelled SYNTHETIC |
| **BUG-009** broker certificate for an IP address | **Found and fixed.** The ESP32 TLS library (mbedTLS 2.28.7 in Arduino-ESP32 2.0.17) checks the host only against DNS names. Tested with mbedTLS 2.28.3: the old certificate is rejected for `192.168.43.10` (CN_MISMATCH), the fixed one (IP also written as a DNS name) is accepted. `broker/check_esp32_tls.sh` repeats this check before every flash. A certificate for a DNS name passed for that name. The last negative case (that certificate tried against an IP) did not run because the package download in the check's container failed; it is not needed for the setup |
| Not verified (needs the kit) | the real board and probe, the real Wi-Fi/hotspot, TLS on the real chip, calibration, outage behaviour on the real node, a week in grain |
| Not verified (needs the server) | the public broker with a Let's Encrypt certificate and its root on the ESP32 (HOSTING plan §5) |
