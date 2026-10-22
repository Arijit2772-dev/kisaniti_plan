# Contract spec V1 — shared basics (W01 slice, milestone M0)

22 October 2026 • **Status: FROZEN** (contract version `shared/v1`), under the layer-by-layer gate of CHAT-28. Decisions W01-S01 to W01-S08 in the [decision register](../blueprint/01_DECISION_REGISTER.md). Machine-readable schemas and valid/invalid examples: `kisaniti_implementation/contracts/shared/v1/`. Reference implementation: `kisaniti_implementation/core/src/kisaniti_core/` (`ids`, `quantity`, `timeutil`, `provenance`, `errors`).

This spec fixes the basics every later layer uses: identifiers, tenant scope, exact numbers, time, provenance and errors. It does **not** freeze any layer's business policy (increments, calendars, economic guardrails, storage thresholds, roles). Those stay in their own slices and remain DEMO or UNKNOWN where the design says so.

Changing anything here needs a new contract version (`shared/v2`), updated schemas on both producer and consumer sides, and a decision-register entry.

## 1. Canonical identifiers (W01-S01)

| Rule | Value |
|---|---|
| Form | `<kind>_<ULID>`: a lowercase kind prefix, an underscore, then a 26-character [ULID](https://github.com/ulid/spec) in Crockford base32, upper case |
| Pattern | `^(fpo\|mbr\|lot\|mnd\|cmd\|stl\|usr\|dev\|pln\|rev\|run\|src\|dsv\|pol)_[0-9A-HJKMNP-TV-Z]{26}$` |
| Opaque | Consumers never parse meaning out of an ID except its kind |
| Ordering | Where an order is needed (e.g. the optimizer tie-break), IDs compare as plain byte strings |
| Provider IDs | AGMARKNET market names/IDs, Google place IDs etc. are **never** canonical IDs; they map through versioned mapping records |
| Fixture IDs | Deterministic for reproducibility: time part `0000000000`, random part = first 80 bits of SHA-256(`fixture:<namespace>:<kind>:<key>`). A leading `0000000000` therefore marks a fixture ID; it does not replace the provenance label |

Kinds:

| Kind | Object | Tenant-owned? |
|---|---|---|
| `fpo` | FPO (tenant) | is the tenant |
| `mbr` | Farmer member | yes |
| `lot` | Produce lot | yes |
| `stl` | Storage location | yes |
| `usr` | Staff user | yes |
| `dev` | Sensor device (assignment is tenant-owned) | yes |
| `pln` / `rev` | Selling plan / plan revision | yes |
| `run` | Optimization or job run | yes |
| `pol` | Policy version (FPO-specific; global policies use `fpo_id: null`) | mixed |
| `mnd` | Mandi (market) | no — global reference data |
| `cmd` | Commodity (and variety series) | no — global reference data |
| `src` | Source batch / import | no for market data; yes for FPO records |
| `dsv` | Dataset version | no |

Adding a kind is a contract version change.

## 2. Tenant scope (W01-S02)

1. Every tenant-owned record carries `fpo_id`. Global reference data (mandis, commodities, market observations, MSP schedule) carries no `fpo_id`.
2. The server derives the acting FPO from the authenticated principal. An `fpo_id` in a request body is checked against it, never trusted.
3. A reference to another FPO's object answers **404 `NOT_FOUND`**, exactly as if the object did not exist (no existence leak). A body whose `fpo_id` differs from the principal's FPO answers **422 `TENANT_SCOPE_VIOLATION`**.
4. Pure `core/` functions take the FPO explicitly and reject inputs that mix FPOs (`TENANT_SCOPE_VIOLATION`). They never look up a tenant themselves.
5. Sensor tenant/assignment comes from registered device credentials and server history, never from payload fields (carried forward from the Layer 1–3 handoff §7).

## 3. Exact quantities, money and rates (W01-S03)

All business numbers are **decimal strings** with a fixed number of fractional digits. Binary floats are never used for stock, money or rates. The unit is part of the **field name suffix**, so a value can never travel without its unit.

| Type | Field suffix | Unit | Scale (fractional digits) | Range | Notes |
|---|---|---|---|---|---|
| Quantity | `_kg` | kilogram | 3 (1 g) | 0 to 1,000,000,000 | Canonical stock unit. Never negative |
| Quantity change | `_delta_kg` | kilogram | 3 | −1,000,000,000 to 1,000,000,000 | Only for adjustments/corrections |
| Money | `_inr` | Indian rupee | 2 (1 paisa) | −1,000,000,000,000 to 1,000,000,000,000 | |
| Rate per kg | `_inr_per_kg` | ₹/kg | 4 | −10,000,000 to 10,000,000 | Planning and costs |
| Rate per quintal | `_inr_per_quintal` | ₹/quintal | 2 | −1,000,000,000 to 1,000,000,000 | AGMARKNET and MSP native unit |
| Distance | `_km` | kilometre | 3 (1 m) | 0 to 100,000 | |

**Serialisation.** Pattern `^-?(0|[1-9][0-9]*)\.[0-9]{scale}$`: always exactly `scale` fractional digits, no exponent, no `+`, no leading zeros, and no negative zero (`-0.000` is invalid). Examples: `"1250.000"` kg, `"2389.00"` ₹/q, `"23.8900"` ₹/kg. Producers and consumers both use this one canonical form, so equal values have equal strings (safe for hashing and idempotency).

**Conversions.** 1 quintal = 100 kg exactly. ₹/quintal ÷ 100 = ₹/kg; scale 2 becomes scale 4 with no rounding. UI may display quintals; storage and APIs use the table above.

**Rounding.**
- Quantities are **never** rounded. Input with more fractional digits than the scale is rejected with `PRECISION_EXCEEDED`, not rounded.
- A money amount computed from quantity × rate is rounded **once**, per line, to paise with **round half to even**. A total is the sum of its rounded lines, so every printed total reconciles with its lines.
- Ratios (e.g. farmer fulfilment) are exact fractions. When shown, they travel as `{ "numerator_kg", "denominator_kg" }` plus an optional display string with 6 decimals (round half to even) that is labelled display-only and never compared.

**Out of scope here:** sensor measurements (temperature, relative humidity) have their own precision in the L3b sensor slice; they are finite decimals, never NaN or Infinity. The planning increment and solver normalisation quantum are separate concepts (architecture §9) and belong to the allocation slice.

## 4. Time (W01-S04)

| Kind | Form | Meaning |
|---|---|---|
| Instant | `YYYY-MM-DDTHH:MM:SS.sssZ` (exactly 3 fractional digits, `Z` only) | A technical moment in UTC: received, generated, evaluated, created times |
| Business date | `YYYY-MM-DD` | A calendar date in the FPO's time zone (mandi date, dispatch date, sale date) |
| Time zone | IANA name on the FPO record | V1 allows only `Asia/Kolkata` (UTC+05:30, no daylight saving) |

Rules:
1. Instants with an offset other than `Z`, without fractional seconds, or naive (no zone) are rejected (`INVALID_INSTANT`). Producers emit exactly milliseconds.
2. An instant becomes a business date only by conversion in the FPO's zone (`2026-10-21T19:00:00.000Z` is business date `2026-10-22` in Asia/Kolkata).
3. Market observation dates are the source's reporting date (India), carried as business dates without conversion.
4. Date ranges are inclusive at both ends: `{ "from": "2026-10-22", "to": "2026-10-28" }` covers seven dates.
5. Same-day cutoffs, operating calendars and planning horizons are **not** frozen here (Layer 3/4 policies).

## 5. Provenance and real/synthetic labels (W01-S05)

Every stored record and every result carries a `provenance` object:

| Field | Values |
|---|---|
| `origin` | `REAL` (observed from a real source), `POLICY` (an official policy value such as MSP, CHAT-20), `SYNTHETIC` (generated, fictional or simulated) |
| `source_kind` | `AGMARKNET`, `MSP_SCHEDULE`, `FIXTURE_GENERATOR`, `SIMULATOR`, `SENSOR`, `MANUAL_ENTRY`, `ROUTE_PROVIDER`, `DERIVED` |
| `source_ref` | Free text: file name and SHA-256, URL, generator script, device ID |
| `source_version` | Optional version or revision string |
| `generator` | Required when `origin = SYNTHETIC`: `{ "script", "version", "seed" }` |
| `acquired_at` | Instant |

And every run carries `mode`: `RESEARCH`, `DEMO` or `LIVE`.

Rules:
1. **Taint rule.** A derived object's origin is `SYNTHETIC` if any input is `SYNTHETIC`; otherwise `REAL`. (Example: a plan from real prices and fictional lots is `SYNTHETIC`.)
2. Accuracy and benefit metrics include only `REAL` observations. `POLICY` values are never forecast training rows or forecast accuracy (CHAT-20).
3. Anything `SYNTHETIC` is shown with the DEMO label in the UI and the paper.
4. `LIVE` mode refuses `SYNTHETIC` inputs with `SYNTHETIC_INPUT_IN_LIVE`. No silent fallback from LIVE to DEMO settings.
5. The origin of sensor and organisation data comes from the registered source and execution mode, not from a user-editable flag.

## 6. Unknown values, readiness and errors (W01-S06, W01-S07)

**Unknown is never zero.** A value that may be missing is wrapped:

```json
{ "status": "AVAILABLE", "value": "2389.00" }
{ "status": "UNKNOWN", "reason": "NO_REPORT_ON_DATE" }
{ "status": "UNAVAILABLE", "reason": "SOURCE_DOWN" }
```

`UNKNOWN` = the fact exists but we do not know it (e.g. storage condition without a fresh reading). `UNAVAILABLE` = our source could not supply it. `reason` is an upper-snake-case code.

**Readiness** (for a named purpose, Layer 1–3 handoff §4): `READY`, `READY_WITH_WARNINGS` or `BLOCKED`, with `purpose`, `issues[]`, `policy_version` and `evaluated_at`. The health endpoint uses the same shape: `GET /health/live` (process up) and `GET /health/ready` (readiness of `database` and `migrations` components).

**Errors** use [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) problem details (`application/problem+json`):

```json
{
  "type": "https://kisaniti.invalid/errors/PRECISION_EXCEEDED",
  "title": "Precision exceeded",
  "status": 422,
  "code": "PRECISION_EXCEEDED",
  "detail": "quantity_kg has 4 fractional digits; at most 3 allowed",
  "instance": "/api/v1/...",
  "request_id": "...",
  "issues": [{ "code": "PRECISION_EXCEEDED", "path": "/lots/0/quantity_kg", "message": "..." }]
}
```

| Code | HTTP | When |
|---|---|---|
| `INVALID_REQUEST` | 400 | Unparseable body, wrong content type |
| `VALIDATION_FAILED` | 422 | Schema or business validation failed (details in `issues`) |
| `INVALID_DECIMAL` | 422 | Not a canonical decimal string |
| `PRECISION_EXCEEDED` | 422 | More fractional digits than the type allows |
| `OUT_OF_RANGE` | 422 | Outside the type's range |
| `UNIT_MISMATCH` | 422 | Arithmetic or comparison across units |
| `INVALID_ID` | 422 | Not a canonical ID, or wrong kind |
| `INVALID_INSTANT` / `INVALID_DATE` | 422 | Bad time value |
| `TENANT_SCOPE_VIOLATION` | 422 | Body or input mixes FPOs |
| `SYNTHETIC_INPUT_IN_LIVE` | 422 | Synthetic data in LIVE mode |
| `UNAUTHENTICATED` | 401 | No valid login |
| `FORBIDDEN` | 403 | Logged in, not allowed |
| `NOT_FOUND` | 404 | Missing, or belongs to another FPO |
| `CONFLICT` | 409 | State conflict (e.g. revision changed) |
| `IDEMPOTENCY_KEY_REUSED` | 409 | Same `Idempotency-Key`, different request body |
| `NOT_READY` | 503 | A dependency's readiness is `BLOCKED` |
| `DEPENDENCY_UNAVAILABLE` | 503 | External provider or database unreachable |
| `INTERNAL` | 500 | Unexpected failure (logged with `request_id`; no private data in the message) |

Every failure maps to one of these codes; no silent `except: pass`. Layers may add codes in their own slices (with a version bump of that slice).

## 7. Versioning (W01-S08)

- JSON Schemas use draft 2020-12 with `$id` `https://kisaniti.invalid/contracts/<slice>/v<n>/<name>.schema.json`. (`.invalid` is a reserved domain: the IDs are names, not URLs to fetch.)
- Top-level documents carry `"contract": "<name>/v<n>"`.
- Producer and consumer tests load the same schema files from `kisaniti_implementation/contracts/`. CI validates every example: each file in `examples/valid/` must pass and each in `examples/invalid/` must fail.

## 8. What is still open (not frozen by this slice)

Planning increments, operating calendars and horizons; economic guardrail; roles and permissions; idempotency storage details (W02); storage thresholds (L3b); market observation and MSP_SCHEDULE schemas (L1 slice); sensor event identity and time quality (L3b slice). Each needs its own slice before its product code starts.
