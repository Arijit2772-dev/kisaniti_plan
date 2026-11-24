# L3a Model registry contract v1 (`models/v1`), 24 Nov 2026

**Status:** CONFIRMED for the evidence it carries (C.0 test season, protocol frozen 27 Oct 2026 with the owner's approval, research commit `864005d`; result commit `2444bea`). DESIGN DECISION for the product surface: FULL_PRODUCT_PLAN Phase 1 item 4.

## Purpose
The product shows which price forecast its plans use for each variety and primary horizon, and the frozen evidence for that choice. This is read-only: the registry never trains, runs or chooses a model at request time.

## Source and rule
- Built by `scripts/make_model_registry.py` from `rp:results/C0_protocol/test_season_inference.csv` and `test_run.json`. The file's SHA-256 and the last commit that changed it are recorded. Nothing is typed by hand.
- Protocol §8: a model is BETTER than naive only if all of these hold: Holm-adjusted p < 0.05; the bootstrap interval lies entirely below zero; and the MAE reduction is at least the smallest detectable gain. Otherwise it is NO_DETECTABLE_DIFFERENCE. `status` follows `verdict` exactly: PROMOTED iff BETTER, which the schema enforces.
- Protocol §9: for each variety and primary horizon, the product uses the model that meets §8 (`in_use_reason` PROMOTED). If none does, it uses `naive`, and `in_use_reason` is BASELINE_ENOUGH.
- Refusals (the script stops; no document is produced):
  - more than one BETTER model at one variety and horizon, because the protocol names no winner between non-naive models (UNKNOWN until decided);
  - any other verdict;
  - a horizon that was run but not tested and has no treatment in protocol §4.
- Untested horizons are listed with their treatment from protocol §4: 28 days SECONDARY, 60 days DESCRIPTIVE, with no claims.

## Interface
- `GET /api/v1/models/registry` needs a signed-in user with `data.read`. It returns `models.registry/v1` (`contracts/models/v1/registry.schema.json`), identical for every FPO; it holds no FPO data.

## Result carried by v1
42 of 42 comparisons give NO_DETECTABLE_DIFFERENCE. For 1121 and Basmati 1509 at 1, 7 and 14 days, the model in use is naive with reason BASELINE_ENOUGH.

## Not in v1
- Model choice for crops or horizons outside the protocol.
- Re-running models in the product.
- Any change to the plan policy label `PLAN_USES_NAIVE_DEMO_V1` (plan/v1). That label stays as it is until a separate decision.
