# Slice 7 scope — the one-command chain (X.1, I.1)

26 October 2026 • **Status: FROZEN** on the owner's approval ("Yes, start slice 7", 26 Oct). Recorded as CHAT-48. This slice adds **no new API and no new contract**; it adds one program that uses the frozen contracts (`shared` to `sales`) from the outside, the way a user of the website's API would.

## 1. What it is

`scripts/chain.py` (Python standard library only) run against a running stack (`docker compose up -d --wait`):

```bash
python3 scripts/chain.py --api http://localhost:8000
```

It creates two new fictional FPOs and runs the five stages of the product flow end to end, checking every step against the contract **and against an independent recomputation** (by hand-written Decimal arithmetic and hand-written expected texts, not by calling the product's own functions). It stops at the first broken step, prints the step, what was expected, what happened and the request ID (the trace key of BUG_TRACEABILITY §2), writes a JSON report, and exits non-zero. A complete pass exits 0.

## 2. The chain (each line is a named step)

1. **preflight**: the API is ready; every contract version is listed.
2. **fpos**: two FPOs (A, B) and their admins sign in.
3. **data**: the price dataset exists (the CI fixture is imported only if none exists); a forecast is available with its "not validated" label; at least two eligible mandis.
4. **stock**: three members (Punjabi, Hindi, English), lots of one variety plus one of another, exact registered balances; a second FPO's lot.
5. **sensor**: a simulated device is registered, placed in grain, fed 360 readings through the real ingestion code, assessed by the real job, linked to a lot.
6. **plan**: a proposal from the forecast, mandi basis, entered transport and the optimiser; independent recomputation of price per kg, of the net return per line and of the totals; the sensor lot carries the assessed category, the others UNKNOWN; an idempotent replay returns the same plan; nothing is reserved.
7. **review**: an invalid edit is refused; a valid edit becomes revision 2.
8. **approval**: approval without acknowledging every warning is refused; approval reserves exactly the edit's quantities; a second approval is refused; the free stock afterwards cannot be planned again.
9. **messages**: contact and consent for two farmers, none for the third; the instructions are prepared, the third is blocked and is in the queue; each prepared text contains the exact quantities, the farmer's name and a date with the month name of that farmer's language; the preview masks the phone and says nothing was sent; the third is fixed and re-prepared; a reply is stored and changes nothing.
10. **sales**: a sale that follows the plan consumes the reservation; a second one; an off-plan sale beyond the free stock is refused with `RECONCILIATION_REQUIRED` and then accepted with an explicit release; a void; the execution and the report are recomputed independently (weighted price, values, shares); outcomes keep fictional rows apart.
11. **close**: the remaining reservation is released; a cancellation is prepared only for farmers who were previewed, and in their language.
12. **ledger-audit**: for every lot of A the ledger is read event by event and the balances are recomputed independently and compared with the API; no balance is negative; the reserved stock of every lot equals what the plans' execution views say remains; the sum of the sales equals the report's actual value.
13. **isolation**: FPO B sees none of A's objects (404 or empty everywhere) and its own lot is untouched.

## 3. Proof that it can fail (required, recorded in the report)

- `--sabotage ledger`: before the audit the program writes one stock event **straight into the database** (bypassing the API) and must then fail at **ledger-audit**, naming the lot. CI runs this and requires a non-zero exit and that step name.
- A real bug introduced into the backend (a deliberate, temporary mutation) must make the chain fail at the step it breaks; run by hand and recorded.

## 4. Deliberately not in this slice

A browser-driven run of the pages (the browser-test tool is the owner's open decision K.8); the whole-system backtest against simple practice (X.2: needs real data and the frozen protocol); a run against a public host; real WhatsApp; a load or soak test; deleting the data a run creates (fictional FPOs stay in the stack, named so they are recognisable).

## 5. Rules

- Standard library only; the program never prints a password or a phone number; fictional data only; the operator steps (creating an FPO, importing the fixture, feeding the simulator, running the assessor, the sabotage) use `docker compose exec` and are the only things that need Docker.
- A step's check must be able to fail: every check compares against a value computed in the script from first principles or from the contract, never against what the same endpoint returned a moment before (except to prove idempotence).

## As built (26 Oct 2026)

Two deviations from the plan above, neither changes the scope: the simulator feed runs on the machine that runs the chain (the backend image has no `jsonschema`), reaching PostgreSQL on the published port, as `scripts/demo_storage.sh` does; and the chain adds a VIEWER account and two rounding-tie sales, which the first design lacked. Evidence: product repo `docs/reports/2026-10-26_slice7_chain.md`.
