# The product as a flow: five stages, every step, and what really exists

26 October 2026 • Written after the owner's question whether the five stages are implemented: **(1) collect data from all sources and clean it, (2) make the opinion (forecast, advice, plan), (3) the manager verifies or not, (4) send to farmers on WhatsApp, (5) get feedback.** This is the same work as the layer roadmap ([ROADMAP_ALL_STEPS.md](ROADMAP_ALL_STEPS.md)) cut the way the product is used. It is checked against the product's actual files: tables, API routes, pages.

## What exists and what does not (verified in the code, 26 Oct)

| In the product today | Not in the product at all |
|---|---|
| Login, roles, FPO separation (two walls), audit log | A real WhatsApp send and its delivery webhook; hold-or-sell and the real forecast comparison |
| Devices, sensor readings, hourly storage assessments with persisted hysteresis | Forecasts (the imported prices are shown, not yet used) |
| The fair-allocation algorithm as a library (exact solver, brute-force check, validator) | A plan, a proposal, an approval, a reservation |
| Members, lots and the stock ledger; mandi prices (6 678 real rows, unchecked) and MSP; pages: sign-in, change password, users, status, home, members, lots, prices, storage sensors | Messages, WhatsApp, replies, sales, reconciliation |
| Five CI jobs, a bug register, simulator and conformance checks | The pages for stages 2 to 5 |

So the foundation is in place, and **stages 2, 3, 4 and 5 are not implemented**. Stage 1 now holds sensor readings, members, lots, the stock ledger, MSP and the imported price dataset (slice 1, 26 Oct); the price dataset waits for the owner's hand check before it counts as validated. The research repository holds the price data, the cleaning scripts and the experiments, but none of that is wired into the product yet.

**44 steps: done 29, started 7, not started 7, waiting 1.** (After slice 5, 26 Oct.)

| Stage | Steps | Done | Started | Not started | Waiting |
|---|---|---|---|---|---|
| 1. Collect data from all sources and clean it | 10 | 6 | 3 | 0 | 1 |
| 2. Make the opinion: forecast, advice and a fair plan | 9 | 6 | 1 | 2 | 0 |
| 3. The manager verifies or not | 7 | 6 | 1 | 0 | 0 |
| 4. Send the instruction to the farmers on WhatsApp | 8 | 5 | 1 | 2 | 0 |
| 5. Get feedback and close the loop | 7 | 5 | 1 | 1 | 0 |
| X. Across all five stages | 3 | 1 | 0 | 2 | 0 |

## Stage 1: Collect data from all sources and clean it

Pull prices, rules, farmers' lots and sensor readings; check them; keep what is trustworthy and say what is not. (Layers 1 and 2 (work packages W03 to W05, W08))

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S1.1 | Members of the FPO (name, village, preferred language): contract, tables, API, page | done | AI | org/v1 frozen; tables, API, page in both languages; no phone numbers until consent (slice 4) |
| S1.2 | Lots: crop, variety, declared quantity in exact kilograms, harvest date, shed, link to the storage probe | done | AI | all fields, places, placements and the sensor link (plan/v1) |
| S1.3 | Stock ledger: every movement in exact kilograms, free / reserved / sold, append-only | done | AI | append-only events, derived balances, database-enforced no-negative, idempotency; reservations and sales endpoints come in slices 3 and 5 |
| S1.4 | Mandi master data and distance or transport-cost inputs | started | AI | 86 mandis imported with the coordinates the release holds; distance and transport-cost inputs not built (transport cost UNKNOWN) |
| S1.5 | MSP table inside the product (the values are verified, the table is not imported) | done | AI | 18 verified values imported (checksum-guarded) and shown on the prices page |
| S1.6 | Import of the cleaned price dataset with provenance and exact row reconciliation | done | AI | 6 678 rows imported with exact reconciliation; status PENDING_HAND_CHECK until the owner's hand check; refusals tested |
| S1.7 | Live price source (AGMARKNET or data.gov.in), bounded and authorised | waiting / blocked | BOTH | registration on hold by your decision; monthly files are used meanwhile |
| S1.8 | Cleaning pipeline as a product job (parser, flags, index), not only research scripts | started | AI | research scripts and tests exist; the product job does not |
| S1.9 | Sensor readings: validated, de-duplicated, stored, assessed, shown by device | done | AI | H-1 to H-6; the page exists |
| S1.10 | Data-readiness page: what is fresh or stale, per source and mandi | started | AI | prices page shows dataset status, counts, newest date and its age; per-source and per-mandi staleness not built |

## Stage 2: Make the opinion: forecast, advice and a fair plan

From prices, forecasts, storage condition and stock, produce a recommendation and a plan across the farmers, with its reasons. (Layer 3 (W07 to W10))

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S2.1 | Forecast contract slice (ranges beyond 7 days, 'no reliable signal') | done | AI | plan/v1 frozen (CHAT-44). The rule for the words "no reliable signal" is UNKNOWN until EXP-04; forecasts say NOT_ASSESSED |
| S2.2 | Baseline forecast in the product, with prediction ranges | done | AI | naive and 5-day average with empirical 80% and 95% ranges, labelled not validated; index equals the research script exactly (433 and 369 days) |
| S2.3 | Model comparison on the pre-registered protocol | not started | BOTH | needs your protocol approval and the hand check |
| S2.4 | Storage-risk estimate per lot's probe | done | AI |  |
| S2.5 | Hold-or-sell rule and advice (storage loss, holding cost, interest) | not started | AI |  |
| S2.6 | Fair allocation: exact solver, brute-force check, independent validator | done | AI | the existing optimiser and independent validator run inside the product; proposal checked against the brute-force oracle in a test |
| S2.7 | Planning orchestration: freeze the inputs, run the optimiser, store the proposal with provenance | done | AI | POST /plans freezes the inputs, runs HiGHS, validates, stores an immutable proposal; not-proven results are stored but not proposed |
| S2.8 | Plain-language explanation of a plan (fairness, options excluded and why) | done | AI | coded explanation translated in Punjabi and English; no prose from the backend |
| S2.9 | Pages: prices and forecast, advice, new plan | started | AI | plan list, new-plan form with the price outlook, plan detail done; hold-or-sell advice page not built |

## Stage 3: The manager verifies or not

The FPO manager reviews, edits, approves or rejects; approval reserves the stock exactly and cannot be done twice. (Layer 4 (W12 part 1))

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S3.1 | L4 contract slice: proposal states, revisions, decisions, reservation | done | AI | approval/v1 frozen (CHAT-45) |
| S3.2 | Proposal list and detail with warnings | done | AI | plan states, revisions, approval context with every warning |
| S3.3 | Edit a proposal; the independent validator re-checks it before saving | done | AI | manager edit = new immutable revision, accepted only if the independent validator finds no issue |
| S3.4 | Approve with atomic stock reservation, tested for concurrent approvals on PostgreSQL | done | AI | decision and RESERVED ledger events in one transaction; concurrency tested on PostgreSQL (8 parallel approvals, two plans competing for one lot) |
| S3.5 | Reject or request a change with a reason; release a reservation | started | AI | reject with a reason and release of the remaining reservation done; "request a change" by another person is not built (roles UNKNOWN) |
| S3.6 | Only a user with permission to approve may approve; every decision audited | done | AI | plans.approve is needed for approve, reject, release; each decision row has actor, time and request ID; denials are audited by the access layer |
| S3.7 | Pages: review, compare revisions, approve | done | AI | review box: revisions and changes, edit grid, approval with warning acknowledgements, rejection, reservation and release, in Punjabi and English |

## Stage 4: Send the instruction to the farmers on WhatsApp

Each farmer gets the approved instruction, in their language, with their consent, and the manager sees what was delivered. (Layer 5, part 1 (W12 part 2))

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S4.1 | L5 contract slice: instructions, delivery attempts, templates | done | AI | messaging/v1 frozen (CHAT-46) |
| S4.2 | Farmer contact, language, consent and opt-out records (phone numbers masked) | done | AI | contacts (versioned, masked, access log), per-kind consent events, opt-out as a withdrawal; erasure and data-access screens not built, so fictional numbers only |
| S4.3 | Message templates in Punjabi, Hindi and English, versioned and reviewed | started | BOTH | two templates in Punjabi, Hindi and English, versioned files; every variant UNREVIEWED until a native speaker reads it |
| S4.4 | Preview channel: shows the exact text, sends nothing | done | AI | preview channel: exact text, masked recipient, labelled "nothing was sent"; never SENT or DELIVERED |
| S4.5 | WhatsApp channel through Meta's Cloud API, to an allow-list; a real send only on your explicit yes | not started | BOTH | needs your Meta account and template approval |
| S4.6 | Delivery-status webhook, repeat-safe: sent, delivered, read, failed | not started | AI | needs a public HTTPS address |
| S4.7 | Queue of blocked and failed messages for the manager | done | AI | queue of blocked instructions and blocked or failed attempts, with reasons |
| S4.8 | Pages: messages per farmer | done | AI | messages on the plan page, contacts and consent page, messages page, in both languages |

## Stage 5: Get feedback and close the loop

Replies, actual sales and stock go back into the system, so plans can be compared with what happened. (Layer 5, part 2)

| Step | What | Status | Owner | Note |
|---|---|---|---|---|
| S5.1 | Farmers' replies stored and shown to staff; a reply is never treated as acceptance automatically | done | AI | replies entered by staff, stored, shown to staff with messages.send, never interpreted; the webhook source waits for a public HTTPS address |
| S5.2 | Actual sale entry from a J-form by staff, with validation | done | AI | actual sale from the J-form with validation, exact amount, duplicate J-form refused, void as a compensating record |
| S5.3 | Stock reconciliation: planned against actual, release of unused reservations, off-plan sales | done | AI | sales consume the reservation; off-plan sales with the person choosing which reservation to give up (provisional policy, UNKNOWN to validate); release of the rest through approval/v1 |
| S5.4 | Planned-against-actual and fairness report | done | AI | planned against actual per line, plan price against realised price, planned and sold share per farmer, exact; gross values only |
| S5.5 | Forecast accuracy tracked from real outcomes | started | AI | plan price against realised price, fictional and real kept apart; this is not an evaluation of the forecast method (needs protocol C.0) |
| S5.6 | Farmer comprehension and usability feedback (small, with consent) | not started | BOTH | needs ethics approval first |
| S5.7 | Pages: record a sale, reconcile, reports | done | AI | sales page, execution and report on the plan page, outcomes, replies under each message, in both languages |

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

