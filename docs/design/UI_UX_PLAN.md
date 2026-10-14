# UI / UX plan — manager app

15 October 2026 • Planning draft. The visual style is **open**: the user will supply design ideas (see §6). Screens and rules below come from the Layer 4/5 designs; anything new is marked *proposal*.

## 1. Who uses it and where

- **FPO manager/CEO:** reviews and approves plans (Layer 4).
- **FPO staff:** register members and lots, and record sales from J-forms (Layer 5).
- **Farmers do not use this app.** They receive WhatsApp messages.
- **Device (proposal):** mostly a phone in a mandi, shed or office, sometimes a laptop. Design **mobile-first**, working well at 360 px width and on slow 4G.
- **Language (proposal):** UI available in Punjabi (Gurmukhi), Hindi and English, chosen per user. This is separate from each farmer's message language.

## 2. Design principles

1. **Truth first.**
   - Every number shows its unit and source (forecast, MSP, measured, synthetic).
   - Unknown is shown as "Unknown", never as 0 or blank.
2. **Plan ≠ approval ≠ sale.** Different colours, labels and icons for proposed, approved/reserved, sent, sold and released quantities.
3. **Explain before asking.** An approve button appears only next to the exact revision, its warnings and the stock it will reserve.
4. **Plain words.** "Wheat in your shed is getting damp (Medium risk)", not "aw 0.67".
5. **Safe actions.**
   - Confirm before approve and release.
   - Show pending/failed states honestly.
   - Survive refresh and lost network (idempotent).
6. **Accessible:** good contrast, touch targets ≥ 44 px, works with large system fonts, never relies on colour alone.

## 3. Screens

| # | Screen | Main content | Source |
|---|---|---|---|
| S1 | Sign-in | Phone/password or OTP (OI-12) | — |
| S2 | Home dashboard | Stock summary (free / reserved / sold), alerts (storage risk, stale data, failed messages), plans waiting for review | *Proposal* |
| S3 | Members & lots | Search members; lot list with crop, variety, declared quantity, place, storage status | Layer 1 |
| S4 | Lot detail | Quantities history, placement, sensor chart (temp/RH 7–30 days), risk category with reason, linked plans and sales | Layers 1/3 |
| S5 | Prices | Per crop/variety: recent mandi prices, MSP line, forecast with range and "reliability" label, data age | Layer 3 |
| S6 | Hold or sell advice (held wheat, basmati) | Sell-now vs later comparison with range, storage risk and cost of waiting; "no reliable signal" when applicable | *Proposal* (OI-01) |
| S7 | New plan | Pick crop, lots, candidate mandis, dates, full/partial | Layer 3 |
| S8 | Proposal list | Plans with state, warnings, demo label | Layer 4 |
| S9 | Proposal detail / compare | Per-farmer assignments, map of mandis, totals, fairness in plain words, excluded options with reasons, side-by-side revision compare | Layer 4 |
| S10 | Edit proposal | Change mandi/date/quantity; shows the re-checked result before saving | Layer 4 |
| S11 | Approve | Exact revision, stock to reserve, warnings to acknowledge, final confirm | Layer 4 |
| S12 | Messages | Per farmer: language, preview, delivery state, replies; failed or blocked messages first | Layer 5 / WhatsApp |
| S13 | Record sale | Enter or scan J-form number; quantity, price, mandi; "saved for review" vs "finalised" | Layer 5 |
| S14 | Reconcile | Off-plan or over-plan conflicts; choose which reservation to release, with reason | Layer 5 (blocked until policy) |
| S15 | Devices | Sensor nodes, last reading, battery/power, calibration date | Hardware |
| S16 | Reports | Planned vs actual, forecast accuracy, fairness, exports for the paper | Layer 5 / research |
| S17 | Settings | Users and roles, languages, demo/live mode banner | Layers 4/5 |

A visible **DEMO banner** appears whenever any synthetic data is involved.

## 4. Technical choices (proposals, pin in W02)

- **Framework:** Next.js with TypeScript; a component library such as shadcn/ui + Tailwind CSS for consistent, accessible components.
- **Charts and maps:** a chart library for prices and sensor lines; a light map library for mandi locations.
- **Language:** i18n framework with reviewed Punjabi/Hindi strings; Indian number formatting (₹1,23,456) and units in quintals with exact kg underneath.
- **Testing:** UI tests for each critical flow (approve, record sale, release), plus visual checks at phone and laptop widths.

## 5. Usability evaluation

EXP-11: 3–5 participants, tasks on S9–S13, measure time, errors and SUS score, then fix the worst problems and repeat once.

## 6. Waiting for the user's design ideas

Record the user's ideas here before visual design starts:

- style or reference apps;
- colours/branding;
- logo/name styling;
- dashboard priorities;
- anything to avoid.

Until then, agents must not finalise a visual theme.
