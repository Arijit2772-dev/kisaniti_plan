# Start here in a new chat — the final plan

24 October 2026 • Everything needed to continue without the earlier conversation. It states what "final" means, the order of work, the go/no-go gates, the risks with fallbacks, and the message to paste.

## 1. Message to paste into the new chat

> I am continuing the KisanNiti project (a Thapar Institute research paper plus working software and hardware). Three repos sit side by side in ~/Documents: `kisaniti` (design, decisions, tracker), `kisaniti_research_paper` (data, experiments, paper) and `kisaniti_implementation` (the product). Read, in this order: `kisaniti/docs/NEW_CHAT_START.md`, `kisaniti/docs/PROJECT_TRACKER.md` (RESUME POINT), `kisaniti_research_paper/AGENT_HANDOFF.md`, `kisaniti_research_paper/STATUS.md`. Then do the next unticked step of the tracker that I name. Rules: verify everything, never invent data, numbers or references; real data only for accuracy claims; commit only your own files and push; tell me when something needs me (CAPTCHA, accounts, hardware, sending an email, supervisor decisions). First message: tell me the state in five lines and propose the next step.

## 2. What "final" means (acceptance checklist)

Acceptance by a journal or conference cannot be guaranteed; the aim is to remove every avoidable reason for rejection. "Done" means all of this has recorded evidence.

**Paper 1 (Springer conference, LNNS/LNCS style)**
- [ ] Every number is produced by a script from the result files; every reference's DOI has been opened and the claim checked
- [ ] Forecasting compared on a **pre-registered protocol** (at most 6–8 models, fixed splits, a held-out last season, effect sizes with intervals, multiple-comparison correction); gains below the detectable size (about 11–17% with the current five seasons) are not claimed
- [ ] Cleaning validated by a hand check of about 50 rows plus one independent spot check
- [ ] Allocation and fairness: exact solution matches the brute-force reference on small cases; price of fairness and solver times reported
- [ ] Storage and hardware: described as built or simulated exactly as it is; limits stated (a probe does not detect spoilage)
- [ ] Passive voice, students first and the supervisor last, separate citation brackets, elaborate illustrated figures (supervisor), college AI/similarity check re-run and recorded, AI assistance declared per Springer policy
- [ ] Declarations complete; venue, template and page limit confirmed with the supervisor

**Software**
- [ ] One command starts the whole system; CI green on real PostgreSQL; all acceptance tests for the claimed layers pass
- [ ] Flow works on real price data: data → cleaning → forecast with ranges → storage risk → hold-or-sell → fair plan → manager approval (atomic reservation) → WhatsApp test message → sale from a J-form → stock update
- [ ] Mobile-friendly UI, synthetic people only, privacy contract implemented

**Hardware**
- [ ] ESP32 + SHT30 node built, salt-calibrated, outage-tested, running a week in a labelled grain container, readings visible live in the app; Wokwi twin runs for anyone

**Documentation**
- [ ] Layer guides, user manual, deployment and demo guide, hardware build guide, reproducibility guide for every paper result

## 3. Order of work (critical path first)

```text
A data ✅ (current data is the Paper 1 data) ──► B cleaning ──► C forecasting (needs a pre-registered protocol) ──► D hold-or-sell ─┐
   │                                                                                           ├─► I integration ─► H final paper pass ─► submit
   ├─► A.3 MSP table                          E allocation experiments ──────────────────────────┤
   ├─► A.4 long history (CEDA / data.gov.in)  F hardware (kit → calibration → week run) ────────┤
   └─► (parallel) H.1–H.2 paper edit redo + illustrated figures, G product layers ──────────────┘
```

Suggested sessions, one folder each so they never collide:

| Session | Steps | Needs |
|---|---|---|
| 1 Cleaning | B.1, B.3–B.5 | the user's 50-row hand check (B.2) |
| 2 Allocation | E.1–E.5 | nothing |
| 3 Paper edit and figures | H.1–H.2 | supervisor's view of the drafts |
| 4 MSP table and contracts | A.3, G.1 | nothing |
| 5 Forecasting | C.0–C.7, then D | sessions 1 and 4 finished; more history if possible |
| 6 Product layers | G.2–G.8 | frozen contracts per layer; WhatsApp test number for G.6 |
| 7 Integration | I.1–I.4 | everything above |

## 4. Gates (stop and decide)

| Gate | Question | If no |
|---|---|---|
| G1 after cleaning | Do counts reconcile and does the hand check agree with the blank-date rule? | Fix the rule; do not model on unchecked data |
| G2 before forecasting | Is long history available (only if CEDA's site is up and offers a download)? | **Default: no.** Run the small pre-registered comparison; state the power limit; move rankings and the hold-or-sell claim to Paper 2 |
| G3 after forecasting | Does any model beat the baseline by more than the detectable size? | Report that the baseline is hard to beat; this is a valid result |
| G4 before writing results | Do all numbers come from scripts and match the result files? | Do not write them |
| G5 before submission | Supervisor review, checker score, references, declarations, venue rules | Do not submit |

## 5. Risks and fallbacks

| Risk | Fallback |
|---|---|
| CEDA stays unreachable (portal down on 24 Oct) | **Default (user decision): no email, no registration.** Keep Paper 1's scope; any new chat may re-test the portal with `curl -I https://agmarknet.ceda.ashoka.edu.in` and, if it answers, follow `layers/L1_data/CEDA_RUNBOOK.md` (a download needs no token). The email draft and data.gov.in route stay on file but are not to be used unless the user changes their mind |
| Models do not beat naive | Report honestly; the contribution shifts to the pipeline, fair allocation, calibration of ranges and the negative finding |
| Hardware kit delayed | Wokwi twin and simulated scenarios carry Paper 1; real data go in Paper 2 |
| Checker still flags text | Authors rewrite in their own words; declare AI assistance; do not disguise it |
| Usage limits stop sessions | Every phase ends with a pushed commit and an updated tracker; resume from `progress`/STATUS files |

## 6. What only the user can do (priority)

1. ~~Send the CEDA email / register on data.gov.in~~ — **user decision 24 Oct: no emails or new registrations for now; only use CEDA if its website works later** · 2. Hand-check ~50 rows on the AGMARKNET site (B.2) · 3. Show the figure drafts to the supervisor and confirm venue, author emails and corresponding author · 4. Buy the hardware kit · 5. Create the Meta WhatsApp test number · 6. Ask about ethics approval before any human testing · 7. Talk to one Patiala FPO and one arhtiya (Paper 2).
