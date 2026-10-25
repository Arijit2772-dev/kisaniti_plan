# The product as a flow: five stages, every step, and what really exists

26 October 2026 • Written after the owner's question whether the five stages are implemented: **(1) collect data from all sources and clean it, (2) make the opinion (forecast, advice, plan), (3) the manager verifies or not, (4) send to farmers on WhatsApp, (5) get feedback.** This is the same work as the layer roadmap ([ROADMAP_ALL_STEPS.md](ROADMAP_ALL_STEPS.md)) cut the way the product is used. It is checked against the product's actual files: tables, API routes, pages.

## What exists and what does not (verified in the code, 26 Oct)

| In the product today | Not in the product at all |
|---|---|
| Login, roles, FPO separation (two walls), audit log | Members, lots, stock (no table, no API, no page) |
| Devices, sensor readings, hourly storage assessments with persisted hysteresis | Prices, MSP, mandi data, forecasts |
| The fair-allocation algorithm as a library (exact solver, brute-force check, validator) | A plan, a proposal, an approval, a reservation |
| Pages: sign-in, change password, users, status, home | Messages, WhatsApp, replies, sales, reconciliation |
| Five CI jobs, a bug register, simulator and conformance checks | The pages for stages 2 to 5 |

So the foundation is in place, and **stages 2, 3, 4 and 5 are not implemented**. Stage 1 is only done for the sensor readings. The research repository holds the price data, the cleaning scripts and the experiments, but none of that is wired into the product yet.

**44 steps: done 3, started 3, not started 37, waiting 1.**

| Stage | Steps | Done | Started | Not started | Waiting |
|---|---|---|---|---|---|
| 1. Collect data from all sources and clean it | 10 | 1 | 1 | 7 | 1 |
| 2. Make the opinion: forecast, advice and a fair plan | 9 | 1 | 1 | 7 | 0 |
| 3. The manager verifies or not | 7 | 0 | 1 | 6 | 0 |
| 4. Send the instruction to the farmers on WhatsApp | 8 | 0 | 0 | 8 | 0 |
| 5. Get feedback and close the loop | 7 | 0 | 0 | 7 | 0 |
| X. Across all five stages | 3 | 1 | 0 | 2 | 0 |

## Stage 1: Collect data from all sources and clean it

Pull prices, rules, farmers' lots and sensor readings; check them; keep what is trustworthy and say what is not. (Layers 1 and 2 (work packages W03 to W05, W08))

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S1.1 | Members of the FPO (name, village, preferred language): contract, tables, API, page | not started | AI | none of these tables exist |
| S1.2 | Lots: crop, variety, declared quantity in exact kilograms, harvest date, shed, link to the storage probe | not started | AI |  |
| S1.3 | Stock ledger: every movement in exact kilograms, free / reserved / sold, append-only | not started | AI | needed by plans, approval and sales |
| S1.4 | Mandi master data and distance or transport-cost inputs | not started | AI | coordinates exist in the research repo |
| S1.5 | MSP table inside the product (the values are verified, the table is not imported) | not started | AI |  |
| S1.6 | Import of the cleaned price dataset with provenance and exact row reconciliation | not started | AI | needs the L1 and L2 contract slices and research steps B.4, B.5 |
| S1.7 | Live price source (AGMARKNET or data.gov.in), bounded and authorised | waiting / blocked | BOTH | registration on hold by your decision; monthly files are used meanwhile |
| S1.8 | Cleaning pipeline as a product job (parser, flags, index), not only research scripts | started | AI | research scripts and tests exist; the product job does not |
| S1.9 | Sensor readings: validated, de-duplicated, stored, assessed, shown by device | done | AI | H-1 to H-5; the page is H-6 |
| S1.10 | Data-readiness page: what is fresh or stale, per source and mandi | not started | AI |  |

## Stage 2: Make the opinion: forecast, advice and a fair plan

From prices, forecasts, storage condition and stock, produce a recommendation and a plan across the farmers, with its reasons. (Layer 3 (W07 to W10))

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S2.1 | Forecast contract slice (ranges beyond 7 days, 'no reliable signal') | not started | AI |  |
| S2.2 | Baseline forecast in the product, with prediction ranges | not started | AI |  |
| S2.3 | Model comparison on the pre-registered protocol | not started | BOTH | needs your protocol approval and the hand check |
| S2.4 | Storage-risk estimate per lot's probe | done | AI |  |
| S2.5 | Hold-or-sell rule and advice (storage loss, holding cost, interest) | not started | AI |  |
| S2.6 | Fair allocation: exact solver, brute-force check, independent validator | started | AI | library and one smoke experiment done; not in the product |
| S2.7 | Planning orchestration: freeze the inputs, run the optimiser, store the proposal with provenance | not started | AI | the missing link between the library and the manager |
| S2.8 | Plain-language explanation of a plan (fairness, options excluded and why) | not started | AI |  |
| S2.9 | Pages: prices and forecast, advice, new plan | not started | AI |  |

## Stage 3: The manager verifies or not

The FPO manager reviews, edits, approves or rejects; approval reserves the stock exactly and cannot be done twice. (Layer 4 (W12 part 1))

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S3.1 | L4 contract slice: proposal states, revisions, decisions, reservation | not started | AI |  |
| S3.2 | Proposal list and detail with warnings | not started | AI |  |
| S3.3 | Edit a proposal; the independent validator re-checks it before saving | not started | AI |  |
| S3.4 | Approve with atomic stock reservation, tested for concurrent approvals on PostgreSQL | not started | AI |  |
| S3.5 | Reject or request a change with a reason; release a reservation | not started | AI |  |
| S3.6 | Only a user with permission to approve may approve; every decision audited | started | AI | the permission and the audit log exist; decisions do not |
| S3.7 | Pages: review, compare revisions, approve | not started | AI |  |

## Stage 4: Send the instruction to the farmers on WhatsApp

Each farmer gets the approved instruction, in their language, with their consent, and the manager sees what was delivered. (Layer 5, part 1 (W12 part 2))

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S4.1 | L5 contract slice: instructions, delivery attempts, templates | not started | AI |  |
| S4.2 | Farmer contact, language, consent and opt-out records (phone numbers masked) | not started | AI | privacy contract |
| S4.3 | Message templates in Punjabi, Hindi and English, versioned and reviewed | not started | BOTH | needs a native reader |
| S4.4 | Preview channel: shows the exact text, sends nothing | not started | AI |  |
| S4.5 | WhatsApp channel through Meta's Cloud API, to an allow-list; a real send only on your explicit yes | not started | BOTH | needs your Meta account and template approval |
| S4.6 | Delivery-status webhook, repeat-safe: sent, delivered, read, failed | not started | AI | needs a public HTTPS address |
| S4.7 | Queue of blocked and failed messages for the manager | not started | AI |  |
| S4.8 | Pages: messages per farmer | not started | AI |  |

## Stage 5: Get feedback and close the loop

Replies, actual sales and stock go back into the system, so plans can be compared with what happened. (Layer 5, part 2)

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S5.1 | Farmers' replies stored and shown to staff; a reply is never treated as acceptance automatically | not started | AI |  |
| S5.2 | Actual sale entry from a J-form by staff, with validation | not started | AI |  |
| S5.3 | Stock reconciliation: planned against actual, release of unused reservations, off-plan sales | not started | AI |  |
| S5.4 | Planned-against-actual and fairness report | not started | AI |  |
| S5.5 | Forecast accuracy tracked from real outcomes | not started | AI |  |
| S5.6 | Farmer comprehension and usability feedback (small, with consent) | not started | BOTH | needs ethics approval first |
| S5.7 | Pages: record a sale, reconcile, reports | not started | AI |  |

## Stage X: Across all five stages

Prove it works as one chain. (Integration)

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| X.1 | One command runs the whole chain on fictional data and fails on any broken step | not started | AI |  |
| X.2 | Whole-system backtest against simple practice | not started | AI |  |
| X.3 | Identity, tenants, sensors, storage, CI, bug register (the foundation under all stages) | done | AI | login, FPO separation, devices, assessments, five CI jobs |

## How I will build it: one thin chain first, then depth

The risk of building layer by layer is a deep stage 2 and nothing connected. Instead each slice below makes the chain longer, with fictional farmers and the real, cleaned price data, always through contract, tables, API, pages in both languages, tests and a report:

| Slice | Delivers | Steps |
|---|---|---|
| 1 | Members, lots, stock ledger, mandi and MSP reference data, the price import with provenance; the storage page | S1.1 to S1.6, S1.10, H-6 |
| 2 | A first plan: baseline forecast with ranges, storage risk, the existing allocation library run by an orchestration step, stored as a proposal with its explanation | S2.1, S2.2, S2.6 to S2.9 |
| 3 | The manager's review, edit, approval with exact reservation, rejection, release | S3.1 to S3.7 |
| 4 | Messages: contacts and consent, templates, the preview channel, delivery records; the real WhatsApp send once your Meta account is ready | S4.1 to S4.8 |
| 5 | Replies, actual sales, reconciliation, planned-against-actual report | S5.1 to S5.5, S5.7 |
| 6 | Depth: the pre-registered forecasting comparison, hold-or-sell, live price source, farmer feedback tests | S2.3, S2.5, S1.7, S5.6 |
| 7 | The one-command chain and the whole-system backtest | X.1, X.2 |

After slice 5 the whole path from data to feedback runs on fictional farmers; slice 6 replaces the baselines with whatever the evidence supports. Items that need you (hand check, protocol approval, Meta account, Punjabi review, hosting) sit inside slices 1, 4 and 6 and are listed in the roadmap.

