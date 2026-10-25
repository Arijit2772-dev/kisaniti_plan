# L1/L2 slices `org/v1` and `market/v1` — members, lots, stock ledger, and market reference data

26 October 2026 • **Status: FROZEN** (contract versions `org/v1` and `market/v1`) on the owner's approval of slice 1 of the product flow ([PRODUCT_FLOW_STATUS.md](../PRODUCT_FLOW_STATUS.md)). Schemas and examples: `kisaniti_implementation/contracts/org/v1/` and `…/market/v1/`. Builds on `shared/v1` (IDs, exact quantities, time, provenance, errors, tenant scope) and `access/v1` (principal, capabilities). Design sources: Layer 1 specification v0.2 §2–§4, §7; Layer 2 proposal §5–§6.

## A. `org/v1` — tenant-owned (every record carries the FPO; another FPO's object answers 404)

| Object | Rules |
|---|---|
| **Member** `mbr_` | name, village, preferred message language (`pa`, `hi`, `en`), active. **No phone number and no consent data here**: contact details arrive with the consent controls (stage 4), never earlier |
| **Storage location** `stl_` | name, kind (`COLLECTION_CENTRE`, `FARM`, `WAREHOUSE`, `THIRD_PARTY`), optional village and coordinates. A missing coordinate blocks route-based choices for lots placed there; it is never replaced by a default |
| **Lot** `lot_` | one commodity owned by one member; registered quantity positive and exact (kg, 3 decimals); original variety and grade labels kept as supplied; optional harvest date; optional initial placement; `quantity_verified` records who confirmed the weighing and when. The remaining quantity is **never a field**: it is derived (below) |
| **Placement** | append-only history of where a lot is; the current location is the latest. No automatic default location for a real lot |
| **Stock event** | append-only ledger entry (below) |

### The stock ledger (exact; the only source of any quantity)

Quantities are integers of grams in storage and decimal strings with exactly three fractional digits on the wire. Nothing is rounded; more digits are refused (`PRECISION_EXCEEDED`).

| Kind | Effect | Written by |
|---|---|---|
| `REGISTERED` | + registered quantity | lot creation (always the first event of a lot, exactly once) |
| `ADJUSTED` | ± signed quantity, with a reason (`WEIGHING_CORRECTION`, `SPILLAGE`, `MOISTURE_LOSS`, `OTHER`) | staff, this slice |
| `RESERVED` | + reserved | approval (slice 3) |
| `RESERVATION_RELEASED` | − reserved | rejection or release (slice 3) |
| `RESERVATION_CONSUMED` | − reserved, no change to available | a sale that uses a reservation (slice 5) |
| `SOLD` | − on hand | a sale (slice 5) |

`on_hand = REGISTERED + ADJUSTED − SOLD`, `reserved = RESERVED − RESERVATION_RELEASED − RESERVATION_CONSUMED`, `available = on_hand − reserved`, `sold = SOLD`. **Invariants, enforced by the database itself (a trigger that locks the lot, so two writers cannot both pass):** `on_hand ≥ 0`, `reserved ≥ 0`, `available ≥ 0`. A violating event is refused as a whole (`409 CONFLICT`, code in the issue) and nothing is written. A retried request with the same `Idempotency-Key` returns the original result and writes nothing; the same key with a different body answers `409 IDEMPOTENCY_KEY_REUSED`. A client can never supply a remaining total. Only `REGISTERED` and `ADJUSTED` are written in this slice; the other kinds are defined now so later slices add writers, not meanings.

### Endpoints (under `/api/v1`; `data.read` to read, `data.write` to change, CSRF on changes)

`GET/POST /members`, `GET/PATCH /members/{id}`; `GET/POST /locations`, `GET /locations/{id}`; `GET/POST /lots`, `GET /lots/{id}`, `POST /lots/{id}/placements`, `GET /lots/{id}/placements`, `POST /lots/{id}/adjustments`, `GET /lots/{id}/events`. A body naming an FPO, a remaining quantity or a password-like field is rejected.

## B. `market/v1` — global reference data (no `fpo_id`; every FPO reads it, only the import job writes it)

| Object | Rules |
|---|---|
| **Commodity** `cmd_` | the source's commodity name kept as is (for example `Paddy(Basmati)`), a crop group, and whether an MSP applies (basmati: no) |
| **Mandi** `mnd_` | one record per source market name; `town` groups sub-yards of one town **for analysis only, mandis are not merged** (decision open: OPEN_ITEMS O9) |
| **MSP entry** | official support price per crop, grade, marketing season, source link, verified flag; origin `POLICY`, never a training row or forecast accuracy (CHAT-20) |
| **Price observation** | one reported row of the source, with variety label, min, max, modal price (₹ per quintal, exact, any may be missing and then stays missing, never zero), arrivals in tonnes, and the doubts as flags: `date_inferred` (blank-date rule), `arrivals_shared`, and the parser's flags. Origin always `REAL` |
| **Dataset version** `dsv_` | one imported release with its row counts and the SHA-256 of its manifest, and `validation_status`: `PENDING_HAND_CHECK`, `VALIDATED`, `REJECTED` |

**Rules.** (1) The import is all or nothing and refuses a release whose counts do not reconcile exactly (`rows = dated + blank-date`, every file checksum matches its manifest). (2) Rows from a dataset that is not `VALIDATED` are always shown with a visible label and may not be used for any accuracy or benefit claim; a `REJECTED` dataset is hidden from every list except the dataset list. (3) Changing a dataset's status is an operator action recorded in the audit trail. (4) Nothing is ever deleted: a corrected release is a new version. (5) A price is displayed to the cent as exact decimal; the UI may show quintals but never converts silently.

Endpoints: `GET /commodities`, `GET /mandis`, `GET /msp`, `GET /prices?mandi_id&commodity_id&from&to&variety&limit`, `GET /prices/summary` (rows per season and variety, newest date, staleness), `GET /datasets`.

## C. Not in these slices

Phone numbers, consent and contact data (stage 4); reservations and sales as writers (slices 3 and 5); transport quotes and routes; the live price source (blocked by your decision); sub-yard merging; forecasts; any plan.
