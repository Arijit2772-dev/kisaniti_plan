# Hold-or-sell advisory — Layer 3 design (CHAT-24)

15 October 2026 • Revision 0.1 • **Design proposal approved in principle by the user; contract not frozen; not implemented.** It applies to **held wheat and basmati paddy** (CHAT-20). MSP-path crops do not use it, except held wheat during the MSP procurement window (see §3).

## 1. Purpose and boundary

The question it answers for the manager: **"For this held lot, is it better to sell in the next few days, or to keep it for weeks or months?"**

- **SELL_NOW** → the manager starts a normal plan. The existing optimizer chooses mandi, date and quantity within the near-term horizon.
- **HOLD** → no plan now; the advice is re-checked later.

The advisory:

- is **information for the manager**, not a decision;
- never reserves stock, never messages farmers by itself and never sells;
- uses only data available on the day of the advice (no leakage);
- is a separate, pure, deterministic computation with no network or database access, like the optimizer.

## 2. Inputs (all frozen with versions in the evidence record)

| Input | Meaning | Source |
|---|---|---|
| Lot | FPO, lot, owner, crop, variety, exact available quantity (kg) | Inventory snapshot C02 |
| Today's best price `P0` (₹/q) | Best expected net price if sold within the near-term horizon: max over candidate mandis of (latest usable observed or near-term forecast price − transport cost per quintal); for wheat during the MSP window also compare MSP | Market data C03, forecasts C08, transport C07, MSP_SCHEDULE |
| Forecast quantiles `Q_p(h)` | Price at horizon `h` ∈ {14, 30, 60, 90, 120} days for p ∈ {0.025, 0.10, 0.50, 0.90, 0.975} | Promoted model for that crop/variety/horizon (CHAT-22, EXP-03/04) |
| Model status per horizon | PROMOTED / NOT_PROMOTED / UNAVAILABLE | Model registry C14 |
| Storage category | LOW / MEDIUM / HIGH / UNKNOWN for this lot's placement | STORAGE-DEMO-0.1 (C09) |
| Loss rate `ℓ(cat)` (fraction of quantity per 30 days) | Expected quantity/quality loss while held | **Parameter** — see §5 |
| Holding cost `k` (₹/q per 30 days) | Fumigation, bags, handling | **Parameter** — see §5 |
| Money rate `r` (per year) | Cost of waiting for cash (interest) | **Parameter** — see §5 |
| Margin `τ` (₹/q) | Minimum pessimistic gain to recommend holding | **Parameter** — see §5 |

All money in exact decimals; prices in ₹/quintal; horizons in days.

## 3. MSP interaction for held wheat

- **Inside the notified procurement window** (from MSP_SCHEDULE): `P0` = max(MSP, best private net price). Agencies buy at MSP.
- **Outside the window:** MSP is **not** available as a selling option (assumption, OI-04; confirm with the arhtiya). Only private/market prices count.
- **Holding past the window:** the later price comes only from forecasts, never from MSP.

## 4. Computation (per horizon `h`)

Value of holding to `h`, per quintal of today's quantity, for a quantile `p`:

```text
m        = h / 30                                 (months held)
keep(h)  = (1 − ℓ(cat)) ^ m                       (fraction left after loss)
V_p(h)   = Q_p(h) · keep(h)  −  k · m  −  P0 · r · h / 365
G_p(h)   = V_p(h) − P0                            (gain vs selling now, ₹/q)
```

Recommendation rule, in this order:

1. **UNAVAILABLE** if `P0` cannot be computed (no usable price or transport). Give the reason.
2. **SELL_NOW (storage)** if the storage category is **HIGH**. Grain conditions favour spoilage, so holding is not advised whatever the price. This is shown with the risk reason; the manager can still choose otherwise.
3. Keep only horizons with status PROMOTED. If none → **NO_RELIABLE_SIGNAL** ("no trustworthy forecast; sell when convenient").
4. If the 80% range is too wide, i.e. `Q_0.90(h) − Q_0.10(h)` > `w_max · P0` for every kept horizon → **NO_RELIABLE_SIGNAL**.
5. If some kept `h` has `G_0.10(h) ≥ τ`, the pessimistic case still beats selling now → **HOLD_UNTIL(h\*)**. `h\*` is the kept horizon with the largest `G_0.50` among those that pass.
6. If every kept `h` has `G_0.50(h) ≤ 0` → **SELL_NOW (price)**: waiting is not expected to pay.
7. Otherwise → **UNCERTAIN**: holding *may* gain; show `G_0.10` to `G_0.90` and let the manager and farmer decide.

**UNKNOWN storage:** the architecture forbids inventing a rank for UNKNOWN. Compute the rule twice, with `ℓ(LOW)` and with `ℓ(MEDIUM)`.

- If both give the same recommendation, return it with the note "storage unmonitored".
- If they differ, return **UNCERTAIN** with both results shown.

**Re-evaluation:** when a new forecast version is promoted or published (expected weekly), when the lot's storage category changes, or on manager request. Never on every sensor reading.

## 5. Parameters (must be sourced before the paper; DEMO values until then)

| Parameter | DEMO starting value | Status / how to replace |
|---|---|---|
| `ℓ(LOW)` | 0.5% per 30 days | **Assumption.** Replace with a literature value for on-farm wheat/paddy storage losses in North India (EXP-00). Test 0–2% in EXP-07 sensitivity |
| `ℓ(MEDIUM)` | 2% per 30 days | **Assumption.** Same; test 1–5% |
| `ℓ(HIGH)` | not used (rule 2) | — |
| `k` | ₹5 per quintal per 30 days | **Assumption.** Ask FPO/arhtiya (fumigation tablets, bags); test ₹0–20 |
| `r` | 7% per year | **Assumption** close to subsidised crop-loan rates; test 4%, 7%, 12% scenarios. Confirm current KCC rate before the paper |
| `τ` | ₹0 per quintal | **Choice:** hold only if even the pessimistic (10th percentile) case does not lose. Test ₹0, ₹25, ₹50 |
| `w_max` | 0.25 (range wider than 25% of `P0`) | **Choice;** tune in EXP-04/07 |

All parameters live in a versioned `HoldSellPolicy` (no generic settings blob). Each output records the policy version, and the UI shows "DEMO assumptions" while any value is unsourced.

## 6. Output contract — `HoldSellAdviceV1` (semantic, to freeze in W01)

- **Identity and inputs:** FPO, lot, crop/variety, quantity, evaluation date/time, input cut-off; the `P0` value with its source breakdown (mandi, price, transport, MSP flag).
- **Per-horizon table:** quantiles, model status/version, `keep`, cost terms, and `G_0.10/G_0.50/G_0.90`.
- **Storage and settings:** storage category and its assessment reference; policy version and parameter values.
- **Recommendation:** one of SELL_NOW(storage | price), HOLD_UNTIL(h), UNCERTAIN, NO_RELIABLE_SIGNAL, UNAVAILABLE.
- **Explanation:** plain-language reason codes, the DEMO/synthetic labels, and a "not a guarantee" note.

The manager screen (UI plan S6) shows a simple chart: today's price line, and the forecast range at each horizon after costs.

## 7. Tests (all NOT_RUN)

| ID | Scenario | Required result |
|---|---|---|
| HS-T01 | Each rule branch 1–7 with hand-computed fixtures | Exact expected recommendation and gains (exact decimals) |
| HS-T02 | Storage HIGH with strong price rise forecast | SELL_NOW(storage); price gain still displayed |
| HS-T03 | Storage UNKNOWN where LOW and MEDIUM disagree | UNCERTAIN with both results |
| HS-T04 | No promoted model / too-wide range | NO_RELIABLE_SIGNAL, never a guess |
| HS-T05 | Held wheat inside vs outside MSP window | `P0` includes MSP only inside the window |
| HS-T06 | Leakage check | Inputs contain nothing dated after the evaluation time |
| HS-T07 | Determinism | Same frozen inputs → identical output |
| HS-T08 | Parameter sensitivity | Recommendation changes are reported in the EXP-07 tables |
| HS-T09 | Integration | SELL_NOW leads the manager to a normal plan request; no stock reserved and no message sent by the advisory itself |

## 8. Evaluation for the paper (EXP-07)

Compare this rule with five policies:

- sell at harvest;
- hold to a fixed month;
- a forecast-median rule (ignores ranges);
- perfect foresight (upper bound only);
- the farmer's typical practice, if known.

Use past seasons, with only past data at each decision date. Report per season and scenario:

- mean and 5th-percentile net ₹/q;
- regret against perfect foresight;
- how often each policy loses to selling at harvest.

Run the sensitivity analysis over the §5 parameter ranges.
