# KisanNiti UI design brief: every page and what it must do (24 Nov 2026)

For the owner's design sheets (FULL_PRODUCT_PLAN Phase 5). Taken from the pages that exist and work today, plus the parts still to come (marked **NEW**). The design changes the look, never the behaviour. Every function listed here must stay on the page.

## 0. What to send me per page

- One sheet at **phone width (375 px)** and one at **laptop width (1280 px)**. A Figma link, an image or a PDF is fine.
- Draw these states where the page has them: **loading**, **empty** (no data yet), **error** (the server refused; show the message), **no permission** (the user may look but not change).
- Show the **Punjabi** version of one page at least. Gurmukhi text is often longer than English, so every label must wrap and not overflow.
- Colours, fonts, spacing, icons and logo are your choice. I set them once as shared styles, so every page follows them.

## 1. On every page (the frame)

| Element | Must contain |
|---|---|
| Top bar | KisanNiti name/logo (link to Home); the FPO's name; the user's name and role; the language switch **ਪੰਜਾਬੀ / English** (saved on the account); **Sign out** |
| Menu | Home, Members, Lots and stock, Prices, Forecast models, Plans, Contacts, Messages, Sales, Storage sensors, Users (admins only), System status. On a phone it must collapse (for example a menu button or a bottom bar). It appears only once the password has been changed |
| DEMO banner | "Demonstration with fictional data" while the mode is DEMO, visible on every page |
| Footer | **Build abc1234** (the exact version running) |
| Messages | a success or error line after every action (the "flash"); errors are shown as text, never only as a colour |
| Confirmations | every irreversible action asks first (approve, reject, release, deactivate, void, reset password) |
| Numbers | quantities always with exactly 3 decimals in kg (1250.000); money in ₹ with 2 decimals; prices in ₹ per quintal. They are never rounded for looks |
| Tables | on a phone each row becomes a card with "label: value" lines (the pages already do this); nothing may scroll sideways |

Roles (what changes per user): **FPO admin** sees everything and manages users; **Staff** enters data; **Viewer** only looks. Extra rights: *approve plans*, *send messages*, *see phone numbers*. Buttons a user may not use are hidden, and a short "you can look but not change" note is shown.

## 2. Pages

### 2.1 Sign in (public)
Username, password, **Sign in** button (shows "Signing in…" while busy), error "Wrong username or password", "too many attempts, try again in N minutes", language switch, the note "This is a demonstration with fictional accounts". **NEW:** a box showing the public demo login (user name and password of the demo FPO).

### 2.2 Change password
Shown at first sign-in and when forced. Current password, new password with hint ("at least 10 characters; several ordinary words work well"), repeat; errors: not the same, too short, current wrong, too weak; **Save new password**.

### 2.3 Home
"Welcome, {name}", FPO and role, the list "What you can do" (from the user's rights). **NEW (to design):**
- counts: members, lots, kg in stock;
- the newest sensor state (condition and age);
- plans waiting for a decision;
- price data age;
- shortcuts to the next steps.

### 2.4 Users (admin only)
- Table: username, name, role, active or inactive, "must change password".
- Add a user: username (with hint), name, role, the extra rights *send messages* and *see phone numbers*.
- After creating a user or resetting a password, the **temporary password is shown once**, in a box with a "give this to them now" note.
- Per row: reset password, deactivate (each confirms first).

### 2.5 System status (public)
API, database and migrations status, time checked, versions (API, core, contracts, build). **NEW:** broker and worker health, and the age of the newest reading.

### 2.6 Members
- Searchable table: name, village, message language, active or inactive.
- Per row: deactivate or activate.
- Add a member: name, village, language (Punjabi, Hindi, English).
- Empty state.
- **NEW:** open a member to see their lots.

### 2.7 Lots and stock (the busiest page)
- Table per lot: member, crop, variety, registered, on hand, reserved, available, sold (kg); place; storage sensor; condition.
- Per lot:
  - change place;
  - link a sensor;
  - adjust stock (± kg with a reason: weighing correction, damage, other), which never goes below zero;
  - show the stock history (ledger).
- Register a lot: member, commodity, variety, kg (exactly 3 decimals), harvest date, place, "weighing confirmed by me".
- Places (godowns): add a place (name, kind).
- **NEW, hold-or-sell advice per lot** (after approval A1): one of **Sell now (storage risk HIGH)**, **No reliable signal**, or **Not available**, with a one-line reason and the label "information only".

### 2.8 Prices
- Dataset card: name, checked or not checked (with a warning when not), rows, dated rows, rows with no date, flagged rows, newest day and its age, table per season and variety.
- Price browser: filter by mandi and variety, table of prices.
- MSP table: season, crop, grade, ₹ per quintal, source link; "?" when not verified.
- **NEW, upload a new AGMARKNET export** (after approval A2): choose a file, then a result with the rows accepted, refused (with reasons) and flagged. New rows stay "pending" until a person marks the file checked.

### 2.9 Forecast models (built 24 Nov)
- Evidence card: test season, run date, protocol and result versions, file and checksum; the rule in one sentence.
- One block per variety × days ahead (6 blocks): "In use: Naive (last price), because no model was clearly better", and a table: model, status (not promoted), change in error vs naive (%), 95% interval, Holm p, pairs.
- Box: horizons without a test (28 days secondary, 60 days descriptive only).

### 2.10 Storage sensors
- Register a sensor: device ID, source (real sensor, Wokwi simulation, software simulator).
- Place it: in the grain or in the shed air, crop, seconds between readings.
- Per sensor:
  - current placement and last reading time;
  - **condition** (low, medium or high risk, or not known with the reason);
  - 24 h middle humidity and temperature, number of readings used, the insect note, any adjustment;
  - assessed at;
  - **humidity chart over the last 24 hours** with the 65% and 70% limit lines;
  - data origin (simulated or real);
  - stop accepting the sensor (confirm first).
- Fixed label: **"Experimental approximation, not measured spoilage"**. **NEW:** "DHT11 = demo sensor" label for the stop-gap node.

### 2.11 Plans (list)
- Table: created, variety, result (proposed, not proposed, approved, rejected), farmers, lots, planned kg, expected return ₹, open.
- **Make a new plan** form:
  - variety;
  - price outlook (per method: days ahead, date, forecast ₹ per quintal, 80% and 95% ranges or "no range: too few past cases"), with its labels "baseline forecast, not validated" and "placeholder, not advice";
  - mandis to consider (ticks; ineligible mandis shown greyed with the reason), transport ₹ per quintal per mandi;
  - days ahead;
  - selling step in kg;
  - **Make the plan**;
  - errors by code.

### 2.12 Plan detail (review, decision, execution, handover) — the main workflow
In this order, each a section:
1. **Proposal**:
   - status;
   - the "Check before acting" warnings;
   - **What to sell**: farmer, lot, mandi, date, kg;
   - totals: planned, left over, sale value, transport, expected return;
   - share of each farmer's stock planned (the fairness view);
   - lots considered, with storage condition and the reason for each excluded lot;
   - **Why this plan**: a list of plain-language explanations;
   - rules used and the input fingerprint.
2. **Revisions**: list (original and the manager's edits, which is current), and the changes from the previous revision (from kg → to kg).
3. **Edit**: change quantities in whole steps, then "Save as a new revision"; invalid edits are listed.
4. **Approve**:
   - table per lot: to reserve, available now, enough stock yes/NO;
   - a tick for each warning;
   - **Approve revision N and reserve the stock**, which confirms first.
5. **Reject**: reason (price, quantity, market, storage, other), confirm.
6. **Decision**: by whom, when, reason.
7. **Reserved stock**: reserved, released and still reserved, with **Release the reservation** (reason, confirm).
8. **What was actually sold**:
   - status;
   - per line: planned, sold, more than planned;
   - per lot: reserved, used by sales, released, given up for an off-plan sale, still reserved;
   - **Planned against actual**: plan price vs realised price, gap, planned value vs sold value, farmer shares planned vs sold.
9. **Send to farmers from your own WhatsApp**:
   - the question "Send this to N farmers?", with the list of who cannot be sent and why, the unreviewed-translation note, and an "I understand" tick;
   - then one row per farmer: language, **Open WhatsApp**, "I sent it" / "I did not send it";
   - earlier sends;
   - label "the system cannot see whether it was delivered".

### 2.13 Contacts and consent
- Per member: language, phone (masked unless the user may see phones), save a new number.
- Permissions **ADVICE / WHATSAPP / RESEARCH**, each given, withdrawn or none. Record given or withdrawn with how the farmer agreed (in person, paper form, digital form) and the notice version.
- Note: "Use fictional numbers only".
- **NEW:** a consent history list and an access log (who saw which number, when).

### 2.14 Messages to farmers
- Per approved plan: **Prepare the messages** (or cancellation messages).
- Table: farmer, language, status (ready or blocked, with the reason), delivery, **Preview**. The preview shows "Would go to" and the exact text, labelled "PREVIEW — nothing was sent".
- "Needs attention" queue.
- Message texts in 3 languages, each with version and reviewed / not reviewed.
- Replies: list, and add a reply typed in, with the note "a reply is never read as agreement".
- **NEW (Phase 4, real WhatsApp):** **Send through WhatsApp** per message or for all, a delivery status that changes on its own (queued → sent → delivered → read, or failed with the reason), and farmers' replies arriving by themselves.

### 2.15 Sales
- Record a sale from the J-form: lot, the plan line it follows (or "no plan"), mandi, date, kg, ₹ per quintal, J-form number.
- List of sales, with **Void** (reason, confirm).
- Off-plan reconciliation: reserved stock held for a sale outside the plan, release it, record again.
- Outcomes: forecast against real prices (pairs, mean error, mean absolute error, real or synthetic), or "none yet".

## 3. What I need from you, in order of use
1. The frame (§1) and Sign in: everything else builds on these.
2. Plan detail (§2.12), Lots (§2.7) and Messages (§2.14): the demo story runs through them.
3. The rest in any order.

Anything you leave out of a sheet, I keep as it works today and style it to match.
