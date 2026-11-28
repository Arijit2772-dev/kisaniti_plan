# KisanNiti — prompt for a design tool (Stitch, Claude design, Figma AI)

Paste everything below the line into the tool. Ask for each page at phone width (375 px) and laptop width (1280 px).
It is the same content as `UI_DESIGN_BRIEF.md`, written for a design tool. All names and numbers in it are fictional.

---

Design a complete, beautiful, modern web app UI called **KisanNiti** ("farmer policy"). Phone-first (375 px) and laptop (1280 px) versions of every screen.

## What the product is
KisanNiti helps a Farmer Producer Organisation (FPO) in Punjab, India, decide **when, where and how much basmati paddy to sell** for its member farmers. It combines mandi (market) prices, a price forecast, an IoT storage sensor in the grain (temperature and humidity), and the FPO's stock. It proposes a fair selling plan; a manager reviews and approves it; farmers are told by WhatsApp in Punjabi, Hindi or English; real sales are recorded and compared with the plan.

## Who uses it
FPO managers and staff in rural Punjab, often on a mid-range Android phone, sometimes on a laptop in the FPO office. Many prefer Punjabi (Gurmukhi script). Roles: **FPO admin** (everything), **Staff** (enters data), **Viewer** (looks only).

## Look and feel
- Calm, trustworthy, agricultural, not flashy: think "a good bank app for farmers". Clean cards, generous spacing, large touch targets (at least 44 px), clear hierarchy.
- Palette suggestion: deep green (fields) as primary, warm wheat/golden accent, off-white background, dark slate text. Status colours: green = good/low risk, amber = medium, red = high/error, grey = unknown. Never rely on colour alone: always a word or icon too.
- Fonts that render Gurmukhi well (e.g. Noto Sans + Noto Sans Gurmukhi, or Mukta Mahee). Gurmukhi labels are often longer than English: everything must wrap, nothing may overflow.
- Simple line icons. Light mode first; a dark mode is a bonus.
- Numbers are important and exact: quantities always as kg with 3 decimals (1250.000 kg), money in ₹ with 2 decimals, prices in ₹ per quintal. Use tabular figures.
- Tables become stacked cards on a phone ("Label: value" lines). Nothing scrolls sideways.
- Every page has: loading, empty (no data yet), error (message shown as text) and "view only — you can look but not change" states.

## The frame (on every signed-in page)
- **Top bar:** KisanNiti logo/name (link to Home); FPO name ("Patiala Kisan FPO (fictional)"); user name and role; language switch **ਪੰਜਾਬੀ / English**; Sign out.
- **Menu:** Home, Members, Lots & stock, Prices, Forecast models, Plans, Contacts, Messages, Sales, Storage sensors, Users (admins only), System status. On phone: a bottom bar with the 4–5 most used items plus "More", or a menu drawer.
- **DEMO banner** on every page: "Demonstration with fictional data".
- **Footer:** "Build abc1234".
- **Flash message** after every action (success or error).
- **Confirm dialog** before every irreversible action (approve, reject, release, deactivate, void, reset password).

## Screens

### 1. Sign in (public)
Logo and a short line ("Fair selling plans for FPO farmers"); username, password, **Sign in** button with a "Signing in…" state; errors "Wrong username or password" and "Too many attempts, try again in 10 minutes"; language switch; a box "Try the demo: username demo.patiala"; note "This is a demonstration with fictional accounts".

### 2. Change password
Current password, new password (hint: "at least 10 characters; several ordinary words work well"), repeat; errors (not the same, too short, current wrong, too weak); **Save new password**.

### 3. Home (dashboard)
"Welcome, Gurpreet"; FPO and role. Cards: members (24), lots (31), stock (18,450.000 kg); storage sensor state (e.g. "Godown 1 — LOW risk, 4 min ago"); plans waiting for a decision (1); price data age ("newest price 3 days old"); shortcuts: Make a plan, Register a lot, Record a sale. List "What you can do".

### 4. Users (admin only)
Table: username, name, role, active/inactive, "must change password". Form to add a user (username, name, role, extra rights: send messages, see phone numbers). After creating or resetting: a one-time box showing the temporary password with "give this to them now — it will not be shown again". Per row: Reset password, Deactivate.

### 5. System status (public)
Health tiles: API, database, migrations, storage broker, ingestion worker, newest sensor reading age; versions (API, core, contracts, build); "checked at" time.

### 6. Members
Search box; table: name, village, message language (Punjabi/Hindi/English), active. Add-member form. Activate/deactivate. Tap a member to see their lots.

### 7. Lots & stock (busiest screen)
Per lot card/row: member, crop (Paddy Basmati), variety (1121, 1509, 1718), registered / on hand / reserved / available / sold (kg), place (godown), linked sensor and its condition badge. Actions per lot: change place, link sensor, adjust stock (± kg with reason: weighing correction, spillage, moisture loss, other; never below zero), stock history timeline. Register-lot form (member, crop, variety, kg, harvest date, place, "weighing confirmed by me" tick). Places: add a godown (name, kind). Per lot a **hold-or-sell badge**: "Sell now (storage risk HIGH)", "No reliable signal" or "Not available", with one-line reason and "information only".

### 8. Prices
Dataset card: name, "checked" or a warning "not yet checked by a person", rows, flagged rows, newest date and age. Price browser: filter by mandi and variety; table date / mandi / variety / modal price ₹/quintal; a simple line chart. MSP table: season, crop, grade, ₹/quintal, source; "?" when unverified. Upload panel for a new AGMARKNET file with a result summary (accepted, refused with reasons, flagged).

### 9. Forecast models
Evidence card (test season, run date, file checksum) and the rule in one sentence. 6 blocks (3 varieties × 2 horizons): headline "In use: Naive (last price), because no model was clearly better", and a small table: model, status "not promoted", change in error vs naive (%), 95% interval, p-value. Note box for untested horizons.

### 10. Storage sensors (IoT)
Register a sensor (device ID, source: real / simulation). Place it (in the grain or in the shed air, crop, reading interval). Per sensor card: placement, last reading time; big **condition badge** (LOW / MEDIUM / HIGH risk, or UNKNOWN with reason like "settling — 6 h after placing"); 24 h middle humidity and temperature; readings used; insect note; **24-hour humidity line chart** with dashed limit lines at 65% and 70%; "REAL sensor" vs "SIMULATED" tag; "DHT22 demo sensor" tag; button "Stop accepting this sensor". Fixed caption: "Experimental approximation, not measured spoilage".

### 11. Plans (list)
Table: created, variety, result badge (proposed, not proposed, approved, rejected), farmers, lots, planned kg, expected return ₹. **Make a new plan** form: variety; price outlook cards per method (days ahead, date, forecast ₹/quintal, 80% and 95% range bars, labels "baseline forecast, not validated" and "not advice"); mandis to consider (checkboxes, ineligible ones greyed with reason) with transport ₹/quintal each; days ahead; selling step (kg); **Make the plan**.

### 12. Plan detail (the main workflow — make this the showcase)
A vertical stepper or tabs with these sections:
1. **Proposal:** status; "Check before acting" warnings; **What to sell** (farmer, lot, mandi, date, kg); totals (planned, left over, sale value, transport, expected return); **fairness view** — bar per farmer showing share of their stock planned; lots considered with storage condition and why any were excluded; **Why this plan** — plain-language bullet explanations; rules used.
2. **Revisions:** original and manager's edits, which one is current, changes (from kg → to kg).
3. **Edit:** change quantities in whole steps, "Save as a new revision".
4. **Approve:** per lot "to reserve / available now / enough stock yes or NO"; a tick per warning; **Approve revision 2 and reserve the stock** (confirm dialog).
5. **Reject:** reason (price, quantity, market, storage, other).
6. **Decision record:** who, when, reason.
7. **Reserved stock:** reserved / released / still reserved; **Release the reservation**.
8. **Actually sold:** planned vs sold per line; **Planned vs actual** — plan price vs realised price, gap, planned value vs sold value, farmer shares planned vs sold (a comparison chart).
9. **Tell the farmers on WhatsApp:** "Send this to 5 farmers?", who cannot be sent and why, an "I understand" tick; then one row per farmer: language, **Open WhatsApp**, "I sent it" / "I did not send it"; label "the system cannot see whether it was delivered".

### 13. Contacts & consent
Per member: language, phone (masked "+91 ••••• •4321" unless allowed), save a new number. Consent chips **ADVICE / WHATSAPP / RESEARCH** — given, withdrawn or none — with how the farmer agreed (in person, paper form, digital form) and notice version. Note "Use fictional numbers only". Consent history and an access log (who saw which number, when).

### 14. Messages to farmers
Per approved plan: **Prepare the messages**. Table: farmer, language, status (ready / blocked with reason), delivery status chip (queued → sent → delivered → read, or failed with reason), **Preview** (shows "Would go to" and the exact message text in Punjabi/Hindi/English, labelled "PREVIEW — nothing was sent"). **Send through WhatsApp** (one or all). "Needs attention" queue. Message templates in 3 languages with version and reviewed / not reviewed. Replies inbox (from farmers on WhatsApp, or typed in) with the note "a reply is never read as agreement".

### 15. Sales
Record a sale from the J-form (lot, plan line or "no plan", mandi, date, kg, ₹/quintal, J-form number). Sales list with **Void** (reason, confirm). Off-plan reconciliation panel. Outcomes: forecast vs real price (pairs, mean error, mean absolute error), or "none yet".

## Sample (fictional) data to use
FPO "Patiala Kisan FPO (fictional)"; manager "Gurpreet Singh"; farmers "Harjit Kaur", "Balwinder Singh", "Manpreet Kaur", "Sukhdev Singh", "Jasleen Kaur" in villages "Nabha", "Sanaur", "Ghanaur"; varieties 1121, 1509, 1718; mandis "Patiala", "Rajpura", "Samana", "Khanna"; prices ₹3,150–₹4,200 per quintal; lots 500.000–3,000.000 kg; sensor "kn-node-001", humidity 57.4 %, temperature 24.3 °C, condition LOW.

## Deliver
Every screen above at 375 px and 1280 px; one screen (Plan detail or Lots) also in Punjabi (Gurmukhi); the shared components (buttons, badges, cards, tables-as-cards, form fields, confirm dialog, flash message, charts) as a small style guide with colours and type sizes.
