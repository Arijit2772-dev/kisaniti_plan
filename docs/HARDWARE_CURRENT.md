# Current hardware and automatic integration

Latest conversation selection: one ESP32-WROOM-32 development board (38-pin, USB-C, soldered headers, built-in antenna) plus one protected SHT30 I2C probe (PROBOTS PRO6111 reference, nominal 50 cm cable). This supersedes older DHT22/DS18B20/manual moisture-meter and Feather/SHT40 kits.

| Buy/assemble for one demonstration node | Quantity |
|---|---:|
| ESP32 board | 1 |
| Protected SHT30 I2C probe | 1 |
| Full-size 830/840-point breadboards | 2 |
| Male-to-male jumper pack | 1 |
| Male-to-female jumper pack | 1 |
| 4.7 kΩ resistors | 2 |
| Four-position, 2.54 mm breadboard-compatible screw terminal | 1 |
| USB-C data cable matching laptop/adapter ports | 1 |
| 5 V / 2 A USB power adapter | 1 |
| Mounting box/base, cable ties and labels | 1 set |
| Small screwdriver and wire stripper | 1 each |
| Labelled demo bag/container and about 1 kg paddy or wheat | 1 set |

No grain-moisture meter, its 9 V battery, DHT22 or DS18B20 is required by the current design. This is a recorded design selection, not a new price/availability or tested-assembly guarantee. Verify delivered probe wire labels; do not infer board header positions from schematic numbering.

Connections: VCC → 3V3; GND → GND; SDA → GPIO21; SCL → GPIO22. One 4.7 kΩ resistor from SDA to 3V3 and one from SCL to 3V3. Keep the initial supplied cable length. Probe at the demonstration grain; electronics outside the bag. Housing food-contact suitability is unverified; keep demonstration grain separate.

Files: [SVG wiring](handoff/2026-10-14/KisanNiti_Circuit/KisanNiti_Wiring.svg), [PNG](handoff/2026-10-14/KisanNiti_Circuit/KisanNiti_Wiring.png), [KiCad schematic](handoff/2026-10-14/KisanNiti_Circuit/KisanNiti_Circuit.kicad_sch). KiCad uses functional module symbols, not manufacturing footprints; no real electrical-rule/physical acceptance result is established.

Path: sensor → ESP32 → 2.4 GHz Wi-Fi/hotspot → authenticated MQTT/TLS → registered sensor adapter → same SensorReading contract as simulator → cleaning/validation → supported StorageAssessment → planning snapshot.

Five-minute sampling is the selected starting setting. Twenty simulated devices/172,800 events are a load fixture, not a purchase count. One probe measures one monitored point, not every bag/FPO. Firmware framework/version, broker/provisioning, buffering/acknowledgment contract and deployment remain to be finalized/tested. Offline readings preserve original measurement time when trustworthy; unknown time must remain unknown.

Automatic temperature/humidity acquisition is selected. Crop-risk approximation rules, thresholds/durations and calibration are still open. Do not copy the discarded prototype's arbitrary thresholds. Unsupported assessments remain UNKNOWN; that alone does not forbid otherwise admissible allocation. Sensors do not measure sales, exact grain moisture, confirmed spoilage or shelf life.
