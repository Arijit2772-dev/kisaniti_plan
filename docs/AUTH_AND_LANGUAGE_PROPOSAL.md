# Login, FPO separation and Punjabi interface — proposal

26 October 2026 • Follows CHAT-35 (login required) and CHAT-36 (Punjabi interface), both user decisions. Everything below the decision lines is a **proposal** until the owner approves it; nothing here is evidence of how a real FPO works (the Layer 4 role set is a DEMO fixture). It closes OPEN_ITEMS O32 once approved and resolves OI-12 (login) and OI-13 (interface language). Existing constraints it respects: [privacy contract](design/PRIVACY_CONTRACT.md) (staff accounts: hashed password or phone OTP; role-based access; access log; phone numbers masked), [shared contract §2](contracts/CONTRACT_SPEC_V1_shared.md) (tenant scope: another FPO's object answers 404, a body with a different `fpo_id` answers 422), Layer 4 (actor comes from authentication, never from names or email patterns; the backend enforces permissions even if the UI hides a button).

## 1. How FPOs are told apart

1. Every user belongs to exactly one FPO through a membership record (`fpo_id`). A person who works for two FPOs gets two accounts (simplest and safest for now).
2. Signing in creates a **principal**: user, FPO, capabilities. Nothing in a request body or URL can change it.
3. **Two layers of enforcement:** (a) every query takes the principal's `fpo_id` explicitly and filters on it; (b) PostgreSQL row-level security on tenant-owned tables as a second wall, so a coding mistake in (a) cannot leak another FPO's rows. Both are tested with two fictional FPOs that must never see each other's devices, readings, lots or plans.
4. Devices are not users: they authenticate to the broker with their own credentials, and their readings reach an FPO only through the device-assignment history (H-4a).
5. The platform operator (the student team for the demo) can create FPOs and first admins, and by default sees no farmer personal data.

## 2. Sign-in method: options

| Option | For | Against |
|---|---|---|
| **A. Phone number or username + password, hashed (Argon2id), server-side session in an HttpOnly cookie** | no outside provider, no per-login cost, works in tests and CI, consistent with the privacy contract | passwords can be shared or forgotten; recovery must be done by an FPO admin |
| B. Phone OTP over WhatsApp or SMS | familiar to farmers and staff; nothing to remember | needs Meta approval (authentication template) or TRAI SMS registration, costs per message, cannot run in CI, couples login to a vendor |
| C. External identity service (Keycloak, Auth0, Firebase) | features out of the box | new infrastructure or a vendor, against the rule of no unnecessary infrastructure; more to deploy and secure |

**Recommendation: A for the demo and the first pilot**, with a second factor (an authenticator-app code) added later if the pilot needs it, and OTP kept as a possible later option behind the same session layer. Details proposed:

- No public sign-up. The first admin of an FPO is created by the operator with a command; that admin creates the other users.
- Session cookie: `HttpOnly`, `Secure`, `SameSite=Lax`; server-side session records with expiry and revocation; sign-out revokes; state-changing requests protected against cross-site requests.
- Slow, rate-limited sign-in with temporary lock after repeated failures; the same message for "no such user" and "wrong password".
- Passwords checked by length and a list of common passwords, not by composition rules (the length to be fixed when it is built; **not decided here**).
- Recovery: an FPO admin resets a user's password (issuing a one-time temporary password that must be changed); the operator resets an admin.
- Audit log of sign-ins, failures, password changes and every view or export of personal data (privacy contract §6). No passwords, tokens or phone numbers in logs.
- Demo accounts: fictional users in two fictional FPOs, created by a script that prints the passwords once; nothing is committed. A public demo shows only these.

## 3. Roles (DEMO fixture; permissions are checked as capabilities, not role names)

| Role | Can | Cannot |
|---|---|---|
| FPO admin (CEO or manager) | review and approve or reject plans, manage users, see everything of their FPO, see phone numbers | see another FPO |
| Staff (operator, accountant) | enter and correct members, lots and sales; prepare plans; see phone numbers only if given the messaging capability | approve plans or manage users |
| Viewer (board member) | read dashboards and reports | change anything; see phone numbers |
| Platform operator | create FPOs and first admins; see system health | see personal data by default |

Real role names and authorities in a real FPO are UNKNOWN (Layer 1 draft) and must come from the pilot FPO; the capability list is the stable part.

## 4. Punjabi interface

**Decision (CHAT-36):** the manager web interface is available in Punjabi (Gurmukhi) and English. **Proposal:**

- **Language in the address:** `/pa/…` and `/en/…`, so a link opens in the language it was shared in. The first visit picks the language from the browser's preference (Punjabi or English, else English); the choice is remembered; the user can switch on every page. A signed-in user's saved choice is stored in their profile. Language is never inferred from a name, religion or caste.
- **Text in catalogs, one file per language**, with identical keys; a check in CI fails the build if any key is missing, empty or unused. API errors carry codes; the website shows the localised message from the catalog (the backend does not translate).
- **Punjabi text quality:** drafts are written by an AI assistant and **must be reviewed by a native speaker before any pilot**; the review status is recorded. Agricultural and market terms (mandi, quintal, MSP, FPO, lot) are fixed in a glossary agreed with the pilot FPO. Quantities, prices, dates and units are never paraphrased.
- **Numbers:** exact decimals as in the rest of the system. **DECISION:** Western digits (0–9, as in most Punjab price boards and the WhatsApp templates) or Gurmukhi digits (੦–੯). Proposed: Western digits, Indian-style grouping, ₹ sign, to avoid misreading amounts.
- **Fonts:** the system's Gurmukhi font through a font list now; a bundled Gurmukhi font later if devices render it poorly (**DECISION** then, since it adds a file or a download). Line height is raised for Gurmukhi.
- **Phones first:** mobile layout, touch targets of at least 44 px, tested with large system fonts, in both languages.
- **Verification:** screenshots of every page in both languages in the browser tests (D-plan §3), checked for clipping and missing glyphs.
- **Farmer messages** (WhatsApp) are a separate setting per farmer (CHAT-23); template approval for Punjabi and Hindi is still to be verified (OPEN_ITEMS O36).

## 5. Build order (contract first)

1. Approve this proposal; add the access contract slice `L0_ACCESS v1` (principal, user, membership, roles and capabilities, error codes 401, 403, 404, 422, session and audit records) with examples and a schema test.
2. Interface language foundation (no login needed, can start immediately): language in the path, catalogs `en` and `pa`, language switch, catalog check in CI; the status page in both languages.
3. Backend: users, memberships, sessions, audit tables (migration), password hashing, sign-in and sign-out endpoints, the principal dependency, row-level security, the operator command; tests including two FPOs, wrong and repeated passwords, cookie flags, cross-site request protection.
4. H-4a device registry and assignments with `fpo_id` (tenant-scoped endpoints now possible).
5. Sign-in screen (S1) and the application shell in both languages; then the demo pages in the order of the demo plan.

## 6. Decisions needed from the owner

1. Sign-in method: A (recommended), B or C.
2. Roles and capabilities in §3 as the DEMO fixture.
3. Digit style, default language when the browser gives no preference, and whether Hindi is added now or later.
4. Who reviews the Punjabi text (a team member who reads Gurmukhi well, or the pilot FPO).
