# Data model and API plan

Represent a scheduled table as a session and keep participation separate so requests, confirmed seats, and cancellations have explicit rules. The API must enforce the same rules for every client.

## Main entities

| Entity | Principal fields | Constraints |
| --- | --- | --- |
| User | UUID, display name, preferredLocale, timestamps, deletion state | preferredLocale is en or it; local identity is independent of sign-in method and geography; public profile excludes authentication details |
| AuthIdentity | UUID, user ID, method, issuer and subject where applicable, email and verification state | Unique external issuer and subject; an identity belongs to one local user; account linking policy remains to be defined |
| MagicLinkChallenge | UUID, recipient email, token hash, expires at, consumed at, timestamps | No plaintext token stored; atomic single consumption before expiry |
| AuthSession | UUID, user ID, created at, expires at, revoked at | Application login session, separate from a scheduled game session; supports per-device and global logout |
| RefreshToken | UUID, auth session ID, token hash, rotation family, expires at, consumed at, revoked at | Track rotation and detect reuse; invalidate affected session on confirmed reuse |
| Venue | UUID, owner ID, name, type, visibility, public area label, public discovery point, timezone, accessibility notes | Public point required before publishing; only owner can edit |
| VenuePrivateLocation | Venue ID, exact address, exact point | Separate access path; never included by default serialization |
| Session | UUID, host ID, hostParticipates, venue ID, title, format, description, starts at, ends at, timezone, capacity, lifecycle, timestamps | hostParticipates defaults to true; end after start; proposed MVP capacity 2 to 16; host must own the venue |
| Participation | UUID, session ID, user ID, status, requested at, decided at, decided by | Unique session and user; statuses pending, approved, rejected, withdrawn |
| Report | UUID, reporter ID, session ID or user ID, category, details, state, timestamps | Authenticated creation; only operators can inspect reports |

Use foreign keys and indexes on session start time, host, lifecycle, and participation session and status. Add a unique constraint on participation `(session_id, user_id)` and a GiST index on the public discovery point. Explicit database checks cover capacity bounds and time ordering.

Session creation accepts `hostParticipates`, a boolean defaulting to `true` when omitted. Create an approved host participation in the same transaction as the session only when it is true. When it is false, create no host participation and leave all player seats available. Return the selected value in session details so the app can show whether the host plays.

Compute confirmed players from approved participation rows initially; available seats equal capacity minus that count, with no additional subtraction for the host. A cached count can be introduced later only with transactional consistency guarantees.

Host authorization is based on `session.hostId`, not the existence of an approved participation. A host who only provides the table can still edit, publish, cancel, review requests, approve players, and access their venue details. Changes to `hostParticipates` after creation remain an open product decision; define the edit and withdrawal rules before implementing that behavior.

Once a venue has a published future session, changes to its location, visibility, and timezone are blocked. A host creates a new venue or cancels affected sessions first. This prevents a venue edit silently moving an approved game.

## Participation transitions

| Actor and action | Transition | Conditions |
| --- | --- | --- |
| Player requests | No membership to pending | Published, future session with a seat currently available; player is not host |
| Player retries request | Pending or approved remains unchanged | Return existing membership without duplicating a seat |
| Player requests again | Withdrawn to pending | Session remains eligible; a rejected player cannot request again in the MVP |
| Host approves | Pending to approved | Host owns session; published and future; capacity available |
| Host rejects | Pending to rejected | Host owns session and session has not started |
| Player withdraws or leaves | Pending or approved to withdrawn | Own non-host membership before start; changes to the host's playing choice remain undecided |

A cancelled session retains participation records for history, but allows no further request or approval and grants no private address access. Completion also ends private address access. Session status is checked alongside participation status on every protected operation.

## Prevent overbooking

1. Begin a database transaction and lock the session row with `SELECT ... FOR UPDATE`.
2. Recheck the host's authority, lifecycle, current server time, and the membership's current status.
3. Count approved participation inside the transaction. If approving a new member would exceed capacity, return `409 TABLE_FULL`.
4. Update the membership and commit. A retry for an already approved membership returns the existing success.

Every operation affecting capacity or eligibility must acquire that same session lock first: approval, withdrawal, capacity edits, cancellation, and completion. Acquire membership locks after the session lock consistently. This serializes competing actions without relying on a stale count in the mobile app.

## REST endpoints

All routes below use `/v1`, except service health endpoints. The following authentication endpoints are planned contracts; exact social callback routes depend on the selected libraries.

| Method and route | Purpose and access |
| --- | --- |
| `POST /auth/magic-link` | Public, rate-limited request with optional en or it locale for the email template; respond consistently without revealing whether an address exists |
| `POST /auth/magic-link/consume` | Public challenge exchange; consume a valid unexpired token once and issue an application session |
| `GET /auth/:provider/start` | Start Google or Apple sign-in with transaction state; provider must be an explicit allowlisted value |
| `GET or POST /auth/:provider/callback` | Validate the provider response using its supported callback method; complete app return without exposing long-lived credentials in the URL |
| `POST /auth/refresh` | Authenticate with a valid refresh token, rotate it, and issue a new short-lived access token |
| `POST /auth/logout` | Revoke the current authentication session using a valid session credential; repeated logout is safe |
| `POST /auth/logout-all` | Authenticated operation revoking all of the user's authentication sessions |

The app receives the API's application access token, not a Google or Apple token for general API authorization. Account linking endpoints and their ownership checks remain to be specified after the policy is decided. Expired, consumed, or revoked login credentials cannot be exchanged again. Session revocation prevents future refreshes; define whether access tokens are also checked against session state or remain usable until their short expiry before implementation.

Authentication is required for the application routes below unless explicitly noted.

| Method and route | Purpose and access |
| --- | --- |
| `GET /me` | Current user's profile and preferences |
| `PATCH /me` | Update own allowed profile fields, including preferredLocale as en or it |
| `DELETE /me` | Request account deletion; process hosted sessions and memberships safely |
| `GET /locations/search?q=...` | Rate-limited global town or area search through the selected geocoder; do not fix results to a launch country |
| `POST /venues` | Create an owned venue with explicit visibility |
| `GET /venues/mine` | List owned venues, with private details only for their owner |
| `PATCH /venues/:id` | Update an owned venue subject to published-session restrictions |
| `GET /sessions` | Nearby published sessions with public fields and availability |
| `POST /sessions` | Create a draft with hostParticipates defaulting to true; create an approved host membership only when the host plays, in the same transaction |
| `GET /sessions/:id` | Public detail for published sessions; drafts only for their host |
| `PATCH /sessions/:id` | Host edits subject to capacity and confirmed-player rules |
| `POST /sessions/:id/publish` | Host validates and publishes a draft |
| `POST /sessions/:id/cancel` | Host cancels an active session; retries return its cancelled state |
| `GET /sessions/:id/location` | Exact location for host or approved participants in an active session |
| `POST /sessions/:id/requests` | Create or return own participation request |
| `GET /sessions/:id/participants` | Host sees requests; approved players see limited confirmed profiles |
| `PATCH /sessions/:id/requests/:requestId` | Host sets a pending request to approved or rejected |
| `DELETE /sessions/:id/participation` | Withdraw own membership idempotently |
| `GET /me/sessions?role=...&status=...` | Hosted, requested, or confirmed sessions, including cancelled history |
| `POST /reports` | Report a session or player for operator review |
| `GET /health/live` and `GET /health/ready` | Public minimal service status; detailed diagnostics restricted |

Define a minimal operator report review and session-disable mechanism before public launch. Implement it as an operator command initially or a role-protected API; a full admin application can follow later.

## Discovery request example

```http
GET /v1/sessions?latitude=45.4642&longitude=9.1900&radiusKm=10&startsAfter=2026-12-01T00%3A00%3A00Z&startsBefore=2026-12-08T00%3A00%3A00Z&format=commander&minAvailableSeats=1&limit=20&offset=0
Authorization: Bearer <access-token>
```

Return session ID, title, format, UTC start and end, timezone, public venue name or area, approximate distance, capacity, confirmed count, and available seats. Include pagination metadata. Nearby discovery must not return private location fields, pending applicants, emails, or identity provider identifiers.

## Contract rules

- Validate DTOs with a global whitelist and reject unknown fields; derive actor and ownership IDs from the authenticated user.
- Return UTC ISO 8601 timestamps and accept explicit offsets or UTC. Resolve daylight saving ambiguity in the creation form before submission.
- Accept valid geographic coordinates worldwide. Interface locale, account location, and device position must not restrict the countries in which a user can create a venue or search for sessions.
- Keep interface locale, session play language, and venue timezone independent. Validate preferredLocale against en and it; use English as the proposed default for magic-link email requests without a locale. Stable error codes allow the app to show localized messages without changing the response schema.
- Return consistent errors with `code`, `message`, optional `fieldErrors`, and a request ID. Use `400` for invalid input, `401` for missing or invalid authentication, `403` for forbidden visible operations, `404` for missing or hidden resources, `409` for conflicting state, and `429` for throttling.
- Make publish, cancel, approve, and withdraw safe to retry. Use a per-user idempotency key on session creation to avoid duplicate sessions when a connection fails after saving.
- Return `Cache-Control: no-store` on protected location responses. Do not put private coordinates in discovery payloads, analytics, or push notification text.
- Generate and review OpenAPI examples for success, full tables, expired sessions, and authorization failures before integrating the mobile client.
