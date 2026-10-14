# Privacy contract — personal data in KisanNiti (CHAT-26)

15 October 2026 • Revision 0.1 • **Design requirement approved by the user; not legal advice; not implemented.** It is based on India's Digital Personal Data Protection Act 2023 and the DPDP Rules 2025 (notified 13 Nov 2025, phased commencement). A lawyer must review this before any real farmer data is used. Sources are listed at the end; re-check them against the official gazette text before a pilot.

## 1. Roles

| Role | Who | Meaning |
|---|---|---|
| Data Principal | Each farmer member; each FPO staff user | The person the data is about |
| Data Fiduciary | The FPO | Decides why and how farmer data is used |
| Data Processor | Whoever runs KisanNiti for the FPO (for the college demo: the student team) | Processes data only on the FPO's instructions |

**College demo rule:** only **synthetic** people. No real farmer's personal data is entered until consent notices, this contract's controls and legal review exist (DEMO/LIVE separation already required).

## 2. Data inventory — collect only what planning needs

| Data | Why needed | Stored? | Notes |
|---|---|---|---|
| Farmer name | Identify member to staff | Yes | — |
| WhatsApp mobile number | Send approved instructions (CHAT-23) | Yes | Shown masked to roles that do not need it; never in logs |
| Preferred language | Message language | Yes | Never inferred from name/religion/caste |
| Village / approximate farm location | Route origin, mandi choice | Yes | Village level by default; exact shed GPS only if the farmer agrees, for routing |
| Lots: crop, variety, quantity, placement | Planning | Yes | — |
| Sales: J-form number, date, mandi, quantity, price | Record outcomes, avoid duplicates | Yes | J-form image optional; keep only the fields needed |
| WhatsApp consent and opt-out records | Messaging permission | Yes | Kept as evidence |
| Sensor readings | Storage risk | Yes | Linked to a placement; becomes personal when tied to a farmer's shed |
| Staff accounts | Login, audit | Yes | Passwords hashed; or phone OTP |
| **Aadhaar number** | Not needed | **Never** | Payment is by the government; KisanNiti does not pay farmers |
| **Bank account, land record numbers, caste, religion, income** | Not needed | **Never** | If ever needed, add a new purpose and consent first |

## 3. Consent and notice

- A short **notice** in Punjabi, Hindi and English, read or shown to the farmer before registration. It covers:
  - what is collected and why (selling advice and instructions, sale records);
  - who sees it (FPO staff by role);
  - how long it is kept;
  - the farmer's rights: access, correction, erasure, grievance, withdrawal of consent;
  - how to complain, including to the Data Protection Board;
  - the FPO's contact.
- **Separate consents** (each recorded with notice version, time, method and the staff member who assisted):
  1. use of data for selling advice and records;
  2. WhatsApp messages;
  3. optional: anonymised use in research/the paper.
- Withdrawing consent must be as easy as giving it. After withdrawal, messaging and planning for that farmer stop. Records the FPO must legally keep are retained but locked from other use.

## 4. Purpose limits

- Use the data only for planning, instructions, sale records and audit.
- No marketing messages.
- No selling or sharing data with traders, arhtiyas or anyone else.
- No profiling beyond planning needs.

**Research use:** only data that is synthetic, or aggregated and anonymised with the separate research consent and college ethics approval.

## 5. Retention and erasure

- Each data type gets a retention period, set with the FPO according to its own legal record-keeping duties.
- **DEMO defaults:**
  - personal contact data is kept while the person is a member plus 1 year, then erased;
  - sale and inventory records are kept with the farmer pseudonymised after that period.
- Erasure means personal fields are removed or irreversibly pseudonymised. Exact inventory totals and audit history stay correct, consistent with the "never silently delete history" rule.
- The large-platform 3-year deletion rule in the Rules targets specified big platforms. It is listed here for awareness, not as an assumed obligation; confirm with legal review.

## 6. Security safeguards

- TLS everywhere, including MQTT.
- Encrypted database storage and backups.
- Hosting in India preferred.
- Tenant isolation (already a core invariant) plus role-based access. Phone numbers are visible only to roles that send messages.
- An access log for personal data views and exports.
- No personal data or tokens in application logs or error reports.
- Secrets in environment variables.
- Least-privilege credentials for devices.
- Regular dependency updates and the CI secret scan ([quality plan](../QUALITY_AND_VERIFICATION.md)).

## 7. Breach response

If personal data may have leaked:

1. Contain the leak.
2. Assess it.
3. Inform the Data Protection Board **without delay**, followed by a **detailed report within 72 hours** of becoming aware (Rule 7; extensions on request).
4. Inform affected farmers without delay through their registered channel (WhatsApp).
5. Record everything.

The FPO, as Data Fiduciary, owns these notices; the processor must alert the FPO immediately. Write a one-page runbook before any pilot.

## 8. Data Principal requests

The app needs a staff screen to:

- show a farmer their data;
- correct it;
- erase or pseudonymise it;
- record a grievance and its resolution.

Each request must be logged with its date and outcome.

## 9. Tests (all NOT_RUN)

| ID | Scenario | Required result |
|---|---|---|
| PR-T01 | Attempt to store Aadhaar/bank fields | Schema rejects; no such columns exist |
| PR-T02 | Register a farmer without the required consent | Messaging blocked; planning blocked if advice consent missing |
| PR-T03 | Withdraw WhatsApp consent | No further messages; past messages unchanged in audit |
| PR-T04 | Erasure request | Personal fields removed/pseudonymised; inventory and sales totals unchanged; audit shows the request |
| PR-T05 | Staff role without contact permission | Phone masked in UI and API |
| PR-T06 | Logs and error reports | No phone numbers, names or tokens appear |
| PR-T07 | Cross-FPO access | Denied (same as L4T01/T07) |
| PR-T08 | Research export | Only synthetic or anonymised and consented records |

## Sources

- [PIB note on DPDP Rules 2025](https://static.pib.gov.in/WriteReadData/specificdocs/documents/2025/nov/doc20251117695301.pdf)
- [KPMG guidance](https://assets.kpmg.com/content/dam/kpmgsites/in/pdf/2025/11/dpdp-rules-2025-guidance-to-dpdp-act-implementation.pdf)
- [Hogan Lovells summary](https://ca.hoganlovells.com/en/publications/indias-digital-personal-data-protection-act-2023-brought-into-force-)
- [Rule 7 text summary](https://dpdpa.com/dpdparules/rule7.html)

These are secondary summaries; verify against the official gazette.
