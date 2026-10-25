# L0 access contract slice v1 — login, sessions, roles, tenant scope

26 October 2026 • **Status: FROZEN** (contract version `access/v1`) under the layer-by-layer gate of CHAT-28, on the owner's approval of CHAT-35, CHAT-36 and CHAT-37. Schemas and examples: `kisaniti_implementation/contracts/access/v1/`. Builds on `shared/v1` ([CONTRACT_SPEC_V1_shared.md](CONTRACT_SPEC_V1_shared.md) §2 tenant scope, §6 errors). Design reasoning: [AUTH_AND_LANGUAGE_PROPOSAL.md](../AUTH_AND_LANGUAGE_PROPOSAL.md). **Values marked DEMO policy are chosen settings, not evidence about any real FPO; they are configuration, not contract.**

## 1. Objects

| Object | Schema | Notes |
|---|---|---|
| `UserV1` | `user.schema.json` | `id` is `usr_<ULID>`; belongs to exactly one FPO (`fpo_id`), except the platform operator (`fpo_id` null). Never carries a password or hash. `origin` labels demo users `SYNTHETIC` |
| `PrincipalV1` | `principal.schema.json` | Who is acting. Built by the server from the session; nothing in a request can set or change it. Carries the CSRF token for this session |
| Requests | `login_request`, `change_password_request`, `user_create_request`, `language_request` | `additionalProperties: false`: a body with `fpo_id`, `password` (on create) or any extra field is rejected |
| `TemporaryPasswordResponseV1` | `temporary_password_response.schema.json` | Shown once on user creation or reset; stored only as a hash |
| `AuditEventV1` | `audit_event.schema.json` | No secrets, no phone numbers, no free text supplied by users |
| `throttled_problem` | `throttled_problem.schema.json` | The one error code this slice adds: `TOO_MANY_ATTEMPTS`, HTTP 429, with `retry_after_s` and a `Retry-After` header |

## 2. Roles and capabilities (DEMO fixture)

Permissions are checked as **capabilities**, never by role name. The capabilities are `users.manage`, `plans.approve`, `data.read`, `data.write`, `contacts.view_phone`, `messages.send`, `platform.admin`.

| Role | Capabilities | Notes |
|---|---|---|
| `FPO_ADMIN` | `users.manage`, `plans.approve`, `data.read`, `data.write`, `contacts.view_phone`, `messages.send` | tenant admin; no `extra_capabilities` |
| `STAFF` | `data.read`, `data.write` plus any of `contacts.view_phone`, `messages.send` listed in the user's `extra_capabilities` | |
| `VIEWER` | `data.read` | no `extra_capabilities`; never sees phone numbers |
| `PLATFORM_OPERATOR` | `platform.admin` | `fpo_id` null; sees no tenant data; created only by the operator command, never over HTTP |

Real role names and authorities in a pilot FPO are UNKNOWN; the capability list is the stable part. A user has one role. A person who works for two FPOs has two accounts.

## 3. Endpoints (all under `/api/v1`)

| Method and path | Needs | Does | Success | Errors |
|---|---|---|---|---|
| `POST /auth/login` | — | checks the password, creates a session, sets the cookie | 200 `PrincipalV1` | 401 `UNAUTHENTICATED` (same answer for unknown user and wrong password), 429 `TOO_MANY_ATTEMPTS`, 422 |
| `POST /auth/logout` | session, CSRF | revokes this session, clears the cookie | 204 | 401, 403 |
| `GET /auth/me` | session | returns `PrincipalV1` (and a fresh view of role, capabilities, language) | 200 | 401 |
| `POST /auth/change-password` | session, CSRF | verifies the current password, sets the new one, clears `must_change_password`, revokes the user's other sessions | 204 | 401, 403, 422 (`WEAK_PASSWORD` issue) |
| `PUT /auth/language` | session, CSRF | saves the interface language (`pa` or `en`) | 200 `PrincipalV1` | 401, 403, 422 |
| `GET /users` | `users.manage` | lists the users of the caller's FPO | 200 | 401, 403 |
| `GET /users/{id}` | `users.manage` | one user of the caller's FPO | 200 `UserV1` | 401, 403, **404 for another FPO's user** |
| `POST /users` | `users.manage`, CSRF | creates a user in the caller's FPO with a server-generated temporary password | 201 `TemporaryPasswordResponseV1` | 401, 403, 409 (username taken), 422 |
| `POST /users/{id}/reset-password` | `users.manage`, CSRF | new temporary password, user must change it, all their sessions revoked | 200 `TemporaryPasswordResponseV1` | 401, 403, 404 |
| `POST /users/{id}/deactivate` | `users.manage`, CSRF | deactivates the user and revokes their sessions; an admin cannot deactivate themselves | 204 | 401, 403, 404, 409 |

A user with `must_change_password` can call only `GET /auth/me`, `POST /auth/change-password` and `POST /auth/logout`; everything else answers 403 with issue code `PASSWORD_CHANGE_REQUIRED`.

## 4. Tenant rules (binding)

1. The acting FPO is `principal.fpo_id`. It is never read from a body, query or path.
2. A reference to another FPO's object answers **404 `NOT_FOUND`**, identical to a missing object (no existence leak), and writes a `CROSS_TENANT_REFERENCE` audit event.
3. A body that names a different `fpo_id` is a schema error here (the field is not allowed); where a later slice allows it, the answer is **422 `TENANT_SCOPE_VIOLATION`**.
4. **Two walls.** (a) every tenant-scoped query takes the FPO explicitly and filters on it; (b) PostgreSQL row-level security on tenant-owned tables, with the API switching to the restricted database role `kisaniti_app` and setting `app.fpo_id` for the transaction of every tenant-scoped request. With no `app.fpo_id` set, the restricted role sees no tenant rows. Pre-principal operations (sign-in, session lookup) use a separate, narrow code path and never serve tenant data.
5. A platform operator has no `fpo_id`, so the restricted role sees no tenant rows for them either.

## 5. Passwords

- Stored as a salted **scrypt** hash with N = 2^17, r = 8, p = 1 (the setting OWASP's Password Storage Cheat Sheet lists for scrypt, read 26 Oct 2026), record format `scrypt$N$r$p$salt$hash`, parameters stored with each hash so they can be raised later. Argon2id is OWASP's first choice; scrypt is used because it is in the Python standard library, which avoids a new dependency and lock-file change. Moving to Argon2id later is a re-hash at next sign-in.
- Policy (DEMO policy): 10 to 256 characters; not equal to or containing the username; not all one repeated character; not in a short built-in list of very common passwords. No composition rules. All Unicode accepted. **A check against a breached-password list is not included** (it needs an outside service); it is future work.
- Temporary passwords are generated by the server (at least 16 characters), shown once, and force a change at first sign-in.
- Verification of an unknown username still performs a hash computation, so timing does not reveal whether a user exists.
- Passwords, hashes and tokens never appear in logs, errors, audit events or responses.

## 6. Sessions, cookie and CSRF

- The session token is 32 random bytes from the system generator; only its SHA-256 is stored. A new session is created at every sign-in (no reuse of an old identifier).
- Cookie `kn_session`: `HttpOnly`, `SameSite=Lax`, `Path=/`, `Secure` unless configured off for plain-HTTP local development. Lifetime (DEMO policy): idle expiry 120 minutes, absolute expiry 12 hours; both configuration.
- Sign-out, password change, reset and deactivation revoke sessions on the server; a revoked or expired session answers 401.
- **CSRF:** every state-changing request (POST, PUT, PATCH, DELETE) other than `POST /auth/login` must carry the header `X-CSRF-Token` equal to the session's `csrf_token`; otherwise 403 `FORBIDDEN` with issue code `CSRF_INVALID`. In addition, when an `Origin` header is present it must match the configured website origin(s).
- The interface language cookie `kn_lang` is separate and holds no identity.

## 7. Throttling

Failed sign-ins are counted per **submitted username** (normalised), whether or not such a user exists, so throttling reveals nothing. DEMO policy: 5 failures in 15 minutes block further attempts for that username for 15 minutes (429 `TOO_MANY_ATTEMPTS`, `Retry-After`). A successful sign-in clears the count. Limits per source address are not used because the address behind a proxy is unreliable; this is a stated limit.

## 8. Audit

Events: `LOGIN_SUCCESS`, `LOGIN_FAILURE`, `LOGIN_THROTTLED`, `LOGOUT`, `SESSION_EXPIRED`, `PASSWORD_CHANGED`, `PASSWORD_RESET`, `USER_CREATED`, `USER_DEACTIVATED`, `LANGUAGE_CHANGED`, `ACCESS_DENIED`, `CROSS_TENANT_REFERENCE`. Each row has the request ID, the actor, the FPO and the subject user; audit rows of an FPO are visible only to that FPO (row-level security) and are append-only. A failed sign-in for an unknown username records no user and no FPO.

## 9. Not in this slice (deliberate)

OTP or SMS or WhatsApp sign-in, second factor, single sign-on, self-registration, password reset by email, one user in several FPOs, API keys for programs, device authentication (devices use broker credentials, see L3B), member (farmer) logins, phone-number storage (comes with the member slice and the privacy controls).

## 10. Test obligations

Contract examples (valid pass, invalid fail); backend tests on real PostgreSQL with two fictional FPOs: no cross-tenant read or write by any endpoint, 404 for another FPO's user, restricted database role sees nothing without `app.fpo_id` and only its own FPO's rows with it; wrong password, unknown user, throttling and unlock; session expiry, revocation and fixation; cookie flags; CSRF missing and wrong; password policy; must-change flow; the operator command; no secret in any log line.
