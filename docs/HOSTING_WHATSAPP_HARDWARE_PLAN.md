# Hosting, URL, WhatsApp and hardware integration — the plan

26 October 2026 • Answers four questions: where it runs and who pays, what the URL is and when it exists, how WhatsApp is connected, and how the hardware connects without rework. Every fact below is tagged **VERIFIED** (read on an official page on 26 Oct 2026), **UNVERIFIED** (third-party or memory; check before relying), or **DECISION** (the owner's). Prices change often; none is quoted from memory. It extends [DEMO_TEST_DEPLOY_PLAN.md](DEMO_TEST_DEPLOY_PLAN.md).

## 1. The recommendation in one view

```text
ESP32 node (farm Wi-Fi) ──MQTT over TLS :8883──►  broker  ◄── kn-ingest ──► PostgreSQL ◄── API ◄── website (HTTPS)
                                                                                  │
 Meta WhatsApp Cloud API ◄── API sends templates ──┘     ── webhook (HTTPS) ──► API (delivery status, replies)
```

Three stages, so nothing waits for an account:

| Stage | Where | Cost | URL | What it is for |
|---|---|---|---|---|
| **1. Now** | the stack on this Mac (Docker), shown to you in the app's browser pane | none | `http://localhost:3000` | build, test, show you everything as it is built |
| **2. Shareable demo and first hardware test** | one small Linux server running the same `compose.yaml`, plus Caddy for automatic HTTPS and Mosquitto for the device broker | see §2; student credit if verified | a real `https://` address (§3) | supervisor and reviewers open it; the real node connects to it; the WhatsApp webhook needs it |
| **3. Pilot** | same design on an owner-controlled or college server, after the privacy contract and a legal review (G.8) | **DECISION** | owner's domain | real FPO, real farmers; not before ethics approval and the DPDP review |

Why a server and not a managed app platform: the device broker needs a plain TCP port (8883) with TLS, which ordinary web-app platforms do not offer; one small Linux server runs the website, API, database and broker together with the files we already have and test in CI (the one-command stack job). **UNVERIFIED:** which managed platforms allow raw TCP; not needed for this design.

## 2. Hosting: what is available (verified where stated)

| Option | What the official page says | Fit | Note |
|---|---|---|---|
| **Azure for Students via the GitHub Student Pack** | **VERIFIED** (education.github.com/pack): 25+ services and a $100 credit for students aged 18+, no credit card required. Duration not stated on that page | **Best first choice**: a small Linux VM for stage 2 | needs you to be verified as a student on GitHub; the credit's validity period must be read in the Azure portal |
| Domain from the pack | **VERIFIED**: Namecheap, one year of a .me domain plus one free SSL certificate; Name.com, a free domain on selected extensions | gives a clean URL | availability of the name you want is not guaranteed |
| DigitalOcean student credit | **UNVERIFIED**: the pack page does not list it and DigitalOcean's student page returned nothing usable; third-party pages give $100 or $200 | do not plan on it | check on DigitalOcean's own page |
| Heroku | **VERIFIED** (pack page): $13 per month for 24 months | website and API only; **UNVERIFIED** whether it can host the broker | not recommended as the main host |
| Oracle Cloud Always Free | **VERIFIED** (Oracle docs): up to 4 Ampere cores and 24 GB in total, but a mobile number and a credit card are needed; unused or over-limit instances can be disabled and deleted; ARM capacity is often unavailable | free, but a card is needed and capacity is unreliable | a fallback only |
| Hetzner | **UNVERIFIED and conflicting**: 2026 articles report price rises and the cheapest plans "currently not available" | do not plan on it | check their price page |
| HiveMQ Cloud (managed broker) | **VERIFIED** (HiveMQ pricing text via search): free Serverless plan, 100 connections, 10 GB traffic a month, MQTT over TLS, no uptime guarantee | an alternative to running our own broker: the node and `kn-ingest` both connect out to it, so the website can live anywhere | telemetry is temperature and humidity, not personal data; fine for the paper and tests; self-host for the pilot |

**Recommendation:** Azure for Students VM (you verify on GitHub and create the Azure account; I cannot create accounts or enter payment details), `compose.yaml` plus Caddy plus Mosquitto on it. If verification fails: the cheapest paid small Linux server you own, or HiveMQ Cloud for the broker with the website elsewhere. **DECISION:** who owns the account (it should be one the team keeps after you graduate; the repository should state this).

## 3. The URL: what it will be, and when

- **Name:** a student-pack `.me` domain if available, otherwise the free DNS name the cloud provider gives the server's public address (Azure public IPs can carry a DNS label; **UNVERIFIED**, check when creating it). Either gives a hostname that Let's Encrypt (through Caddy) can certify, so the site is `https://` with no cost. A bare IP address cannot serve the WhatsApp webhook or a trusted certificate.
- **When it exists:** the same day the server exists. The sequence once you have the account is: create the VM → open ports 80, 443 and 8883 → point the name at it → `docker compose up` with the production override → Caddy obtains the certificate → run the whole-stack script against the URL. I prepare the production override, the Caddy file, the deployment script and the checks beforehand, so your part is only the account and the click-through.
- **After the URL exists (checklist):** (1) set the API and website environment (no secrets in the repo); (2) register Mosquitto users for each device; (3) run `kn-conformance` against the public broker with the software node; (4) run the demo-path script against the URL; (5) set the WhatsApp webhook to `https://<name>/webhooks/whatsapp`; (6) record URL, commit and date in the paper's availability statement; (7) keep the DEMO banner and fictional data only.

## 4. WhatsApp: how it connects

| Item | Fact | Status |
|---|---|---|
| Test setup | A test WABA and test business number are created automatically when you start with the Cloud API; test accounts do not need a payment method on file to send template messages | **VERIFIED** (Meta docs) |
| Recipients on a test number | how many recipient numbers can be added is **not stated** on the page I read | check in the Meta dashboard; plan on your own number only |
| Templates | the only message type allowed outside the customer service window; generally need approval before use | **VERIFIED** |
| Languages (English, Hindi, Punjabi) | whether and how Meta approves Hindi and Punjabi templates was **not found** | **UNVERIFIED**: check the template language list in Meta's dashboard before writing the three-language templates |
| Pricing | per delivered message, by category (marketing, utility, authentication) and recipient country; India is its own market with an INR rate card (files not read); from 1 Oct 2026 utility templates inside an open window are charged; the first 1,000 service messages per number each month are free; messages from users to the business are free | **VERIFIED** (Meta pricing page); **India figures not read** |
| Rate limits | one message per 6 seconds to the same user on average, bursts of up to 45 in 6 seconds, default throughput 80 messages per second per number | **VERIFIED** |
| Webhook | a public HTTPS address that answers 200 quickly; needed for delivery status and replies | **UNVERIFIED** in Meta's own words (third-party guides agree); confirm when implementing |

**Design (so it stays safe and connectable):** the API talks to WhatsApp only through one `MessageChannel` interface with two implementations: `PreviewChannel` (renders the exact text, sends nothing; used in tests, CI and the public demo) and `WhatsAppCloudChannel` (real sends). The real channel needs the access token in an environment variable on the server, never in the repo and never on a public demo; it sends only to numbers on an allow-list that starts with your own number; and **a real send always needs your explicit approval in this session**. Templates are versioned files (language, text, variables) tested by a script that renders them for every language. Consent and an opt-out are part of the privacy contract (G.8); a farmer is never messaged on the basis of this demo.

**What only you can do:** create the Meta developer account and the test app, add your own phone number, create and submit templates, copy the token to the server's environment. I build and test everything up to the channel, and the preview path works with no account at all.

## 5. Hardware: how it connects without rework

The rule from the final plan stands: the backend sees only contract messages. For the real node on a farm:

1. **Broker address and certificate:** the node needs the broker hostname, port 8883 and the CA that signed the broker certificate. With a public certificate (Let's Encrypt) the node must carry the matching root certificate, not our test CA. **UNVERIFIED:** which root the ESP32 TLS stack needs for the current Let's Encrypt chain; the conformance harness against the public broker will show it before the board is ever built.
2. **Identity:** one broker user per device, username equal to the device ID, topic ACLs already limit it to its own telemetry and ack topics (EXISTS). H-4 adds the registry that says which FPO and lot the device belongs to; a device that is not assigned is stored but never used.
3. **Network:** the node joins the farm's Wi-Fi (or a phone hotspot); it buffers 288 samples through outages and resends, as already tested.
4. **Acceptance before trust:** run `kn-conformance publisher` on the real node, then the bench tests (salt calibration, settling time, outage) from the build guide; the website's storage page then shows it with no code change.
5. **Order of the kit** is the longest wait; order it now (F.3).

## 6. Order of work from here, and what each side does

| # | Step | I do | You do | Needs |
|---|---|---|---|---|
| 1 | Verify as a student on GitHub; create the Azure account; read the credit's validity | — | all | your GitHub and college email |
| 2 | Create the Meta developer account and test app; add your phone as a recipient; check the template language list | — | all | your phone number |
| 3 | Order the hardware kit | — | all | money, your choice of seller |
| 4 | H-4 device registry, tenant isolation, per-device credentials | build and test | — | nothing |
| 5 | H-5, H-6 scheduled assessment and the live storage page | build and test | — | step 4 |
| 6 | `MessageChannel` with `PreviewChannel`, templates in three languages as files, tests | build and test | — | contract slice L5 frozen first |
| 7 | Production override, Caddy file, deploy script, whole-stack script | write and test locally | — | nothing |
| 8 | Deploy to the VM, run the whole-stack script against the URL | run | open the ports, point the name | steps 1 and 7 |
| 9 | WhatsApp webhook and one real test message to your number | implement; send only after your explicit yes | token to the server environment, approve the send | steps 2 and 8 |
| 10 | Real node: conformance, bench tests, one-week run | write up | build, flash, run | step 3 |

Steps 1 to 3 are yours and take lead time (verification, template approval, shipping), so start them now. Steps 4 to 7 do not depend on them.

## 7. Risks

| Risk | Effect | Response |
|---|---|---|
| Student verification or the Azure credit fails | no stage 2 server | cheapest paid small Linux server you own, or HiveMQ Cloud for the broker |
| Template approval is slow or a language is not supported | no real WhatsApp send in Hindi or Punjabi | previews in the demo; send English first; report the limit |
| Public broker certificate not accepted by the ESP32 | node cannot connect | found by the conformance run against the public broker first; embed the right root certificate |
| A public demo is abused or misread as real | misleading claims | DEMO banner, fictional data, no tokens on it, rate limits at Caddy |
| Credit runs out | demo goes offline | the compose files run anywhere; the paper's availability statement also archives code and data with a DOI (I.4) |

## 8. Decisions needed from the owner

1. Account owner for the server and the Meta app.
2. Go-ahead to start steps 1 to 3 now.
3. Whether a temporary public link from this Mac (tunnel tool) is wanted before stage 2 (a download and an outward-facing action).
