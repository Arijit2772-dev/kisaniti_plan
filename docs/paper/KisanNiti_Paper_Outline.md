# Research paper outline — KisanNiti

15 October 2026 • **Skeleton only.** Every result marked `[EXP-xx]` must come from an executed run recorded under `kisaniti_implementation/experiments/`. Do not write numbers into this paper before the run exists. Experiment definitions: [research plan](../research/RESEARCH_AND_EXPERIMENT_PLAN.md).

## Working title

*KisanNiti: A Fair, Uncertainty-Aware Selling Decision-Support System for Farmer Producer Organisations — Evidence from Patiala, Punjab*

## Proposed contributions (to be confirmed by results)

1. **Problem framing:** a measured analysis of where selling decisions matter for Punjab paddy/wheat under MSP procurement. Mandi choice matters little for MSP crops; timing and storage matter for held wheat and basmati. [Patiala research](../research/2026-10-15_Patiala_Storage_Risk_and_Unknowns.md), [EXP-01]
2. **Forecast comparison:** baselines, statistical, gradient-boosting and foundation models (Chronos-2) on Punjab wheat and basmati mandi prices at 1–120-day horizons, with interval calibration. [EXP-03, EXP-04]
3. **Fair allocation model:** an exact lexicographic MILP with per-farmer leximin fulfilment. It is checked against a brute-force reference, and its "price of fairness" is compared with weighted-sum and other fairness rules. [EXP-08, EXP-09]
4. **Low-cost storage-risk monitoring:** an ESP32 + SHT30 in-grain water-activity rule, with bench calibration and honest limits. [EXP-06, EXP-13]
5. **End-to-end evaluation:** a historical simulation against simple farmer practice, with negative results reported. [EXP-07, EXP-10]

Design-choice justifications: [design alternatives](../research/DESIGN_ALTERNATIVES.md). Quality rules: [quality and verification](../QUALITY_AND_VERIFICATION.md).

## Sections

1. **Introduction** — Selling problem of small farmers; role of FPOs; research questions RQ1–RQ5 mirroring the contributions.
2. **Background** — MSP procurement, arhtiyas, J-forms, procurement windows and moisture specs in Punjab; basmati private trade; on-farm wheat holding. (Sources: Patiala research note.)
3. **Related work** — [EXP-00]: mandi price forecasting in India; time-series foundation models; fair division/leximin in allocation; grain storage monitoring; agricultural DSS adoption.
4. **System design** — The five layers (data, cleaning, forecasting and optimisation, manager approval, communication and outcomes); separation of facts, predictions, plans and reality; multi-FPO isolation; exact quantities. Figure: architecture diagram. (Source: [architecture](../architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md).)
5. **Methods**
   - 5.1 Data acquisition and cleaning [EXP-01, EXP-02]
   - 5.2 Forecasting models and evaluation protocol [EXP-03–05]
   - 5.3 Storage-risk rule [EXP-06]
   - 5.4 Hold-or-sell advisory [EXP-07]
   - 5.5 Optimisation model with full mathematics [EXP-08, EXP-09]
   - 5.6 Approval/reservation consistency [EXP-11]
   - 5.7 Multilingual WhatsApp instructions [EXP-12]
   - 5.8 Hardware node [EXP-13]
6. **Experimental setup** — Data periods, splits, hardware and software versions, seeds, baselines and statistical tests.
7. **Results** — Planned tables and figures (all empty until runs exist):
   - T1 data coverage by series [EXP-01]
   - T2 cleaning precision/recall [EXP-02]
   - T3 forecast MAE/MASE by crop × horizon with DM-test markers [EXP-03]
   - F1 interval coverage vs nominal [EXP-04]
   - T4 storage-rule agreement [EXP-06]
   - T5 hold/sell policies by season scenario [EXP-07]
   - F2 solver time vs problem size [EXP-08]
   - T6 price of fairness by formulation [EXP-09]
   - T7 end-to-end net ₹/q vs baselines [EXP-10]
   - T8 usability and comprehension [EXP-11, EXP-12]
   - T9 sensor reliability [EXP-13]
8. **Discussion** — When the system helps and when it does not; MSP vs non-MSP crops; adoption and arhtiya relationships.
9. **Limitations and ethics** — Threats to validity (research plan §3); DPDP compliance; consent; synthetic organisation data; single-site study.
10. **Conclusion and future work.**

**Appendices:** full contracts; acceptance test list; reproducibility checklist; consent forms.

## Reproducibility checklist (fill per run)

- Code commit
- Data manifest and checksum
- Package lockfile
- Hardware
- Seeds
- Run time
- Test period untouched until the final run (yes/no)
- Negative results reported (yes/no)
