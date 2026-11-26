# Data model and API plan

Represent a scheduled table as a session and keep participation separate so requests, confirmed seats, and cancellations have explicit rules. The API must enforce the same rules for every client.

## Main entities

| Entity | Principal fields | Constraints |
| --- | --- | --- |
| User | UUID, display name, preferredLocale, timestamps, deletion state | No login username; preferredLocale is en or it; public name does not identify a sign-in account |
| City | UUID, name, country code, canonical discovery centroid | Shared city-level search point; exact session coordinates never drive pre-approval discovery |
| AuthIdentity | UUID, user ID, provider, issuer and subject | Unique external issuer and subject; linking requires explicit ownership proof; do not merge by provider email |
| AuthSession | UUID, user ID, created at, expires at, revoked at | Application login session, separate from a scheduled game session; supports per-device and global logout |
| RefreshToken | UUID, auth session ID, token hash, rotation family, expires at, consumed at, revoked at | Track rotation and detect reuse; invalidate affected session on confirmed reuse |
| Venue | UUID, personal owner ID, name, city ID, timezone, accessibility notes | Only the personal owner can edit; discovery responses expose only the city |
| SessionLocation | Session ID, exact address, exact point, city ID, venue revision | Protected snapshot for every venue type; edits do not relocate unrelated sessions |
| Session | UUID, host ID, hostParticipates, venue ID, title, format, description, starts at, ends at, timezone, capacity, lifecycle, version, timestamps | hostParticipates defaults to true; end after start; proposed MVP capacity 2 to 16; host owns the session |
| Participation | UUID, session ID, user ID, status, requested at, decided at, decided by | Unique session and user; statuses pending, approved, rejected, withdrawn |
| Report | UUID, reporter ID, session ID or user ID, category, details, state, timestamps | Authenticated creation; only operators can inspect reports |
| Notification | UUID, recipient ID, event ID, session ID, aggregate version, type, safe payload, created at, read at | Unique recipient and event; no precise location or credentials in the payload |
| PushDevice | UUID, user ID, installation ID, provider token, platform, enabled state, last seen | Credentials protected; revoke association on logout and disable invalid tokens |
| DeliveryEvent | Event ID, operation ID, recipient IDs, safe payload, delivery state and retry metadata | Durable event-triggered delivery; no periodic notification-table polling |

Use foreign keys and indexes on session start time, host, lifecycle, participation session and status, and notification recipient and event. Add a unique constraint on participation `(session_id, user_id)` and a GiST index on canonical city discovery points. Database checks cover capacity bounds and time ordering.

Session creation accepts `hostParticipates`, a boolean defaulting to `true` when omitted. Create an approved host participation in the same transaction as the session only when it is true. When it is false, create no host participation and leave all player seats available. Return the selected value in session details so the app can show whether the host plays.

Compute confirmed players from approved participation rows initially; available seats equal capacity minus that count, with no additional subtraction for the host. A cached count can be introduced later only with transactional consistency guarantees.

Host authorization is based on `session.hostId`, not approved participation. The host can edit all business fields, including venue, location, format, time, capacity, and hostParticipates, even with approved players present. Toggling false withdraws the host's player membership and frees a seat; toggling true approves it only if resulting capacity permits. Ownership and server-managed IDs are not transferred through an ordinary edit.

Apply each state-changing session or participation operation, its aggregate version increment, affected recipients' notifications, and delivery event records in one database transaction. Preserve existing non-host memberships by default; do not silently evict players when reducing capacity. Reject stale edit versions or an over-capacity result with 409. Session location snapshots or venue revisions allow edits without cancelling the session or moving other games.

## Participation transitions

| Actor and action | Transition | Conditions |
| --- | --- | --- |
| Player requests | No membership to pending | Published, future session with a seat currently available; player is not host |
| Player retries request | Pending or approved remains unchanged | Return existing membership without duplicating a seat |
| Player requests again | Withdrawn to pending | Session remains eligible; a rejected player cannot request again in the MVP |
| Host approves | Pending to approved | Host owns session; published and future; capacity available |
| Host rejects | Pending to rejected | Host owns session and session has not started |
| Player withdraws or leaves | Pending or approved to withdrawn | Own non-host membership before start |
| Host changes playing choice | Approved to withdrawn, or absent/withdrawn to approved | Session owner edits hostParticipates under the session lock; opting in must fit resulting capacity |

A cancelled session retains participation records for history, but allows no further request or approval and grants no private address access. Completion also ends private address access. Session status is checked alongside participation status on every protected operation.

## Prevent overbooking

1. Begin a database transaction and lock the session row with `SELECT ... FOR UPDATE`.
2. Recheck the host's authority, lifecycle, current server time, and the membership's current status.
3. Count approved participation inside the transaction. If approving a new member would exceed capacity, return `409 TABLE_FULL`.
4. Update the membership and commit. A retry for an already approved membership returns the existing success.

Every operation affecting capacity or eligibility acquires that session lock first: approval, withdrawal, host participation edits, capacity or schedule edits, cancellation, and completion. Acquire membership locks after the session lock consistently. This serializes competing actions without relying on a stale mobile count.

## REST endpoints

All routes below use `/v1`, except service health endpoints. The following authentication endpoints are planned contracts; exact social callback routes depend on the selected libraries.

| Method and route | Purpose and access |
| --- | --- |
| `GET /auth/:provider/start` | Start Google or Apple sign-in with transaction state; provider must be an explicit allowlisted value |
| `GET or POST /auth/:provider/callback` | Validate the provider response using its supported callback method; complete app return without exposing long-lived credentials in the URL |
| `POST /auth/:provider/exchange` | Verify supported native SDK credentials with transaction proof; resolve/create the application user and issue the app session |
| `POST /auth/refresh` | Authenticate with a valid refresh token, rotate it, and issue a new short-lived access token |
| `POST /auth/logout` | Revoke the current authentication session using a valid session credential; repeated logout is safe |
| `POST /auth/logout-all` | Authenticated operation revoking all of the user's authentication sessions |

The app receives the API's access token, not a provider token for general API authorization. Callback/exchange routes depend on the selected Google/Apple SDKs; expose only the flows actually used. First verified sign-in creates a profile transactionally under issuer/subject uniqueness. Account linking/unlinking endpoints await policy decisions and fresh identity proof. No local registration, password reset, or recovery code endpoint exists. Provider account recovery belongs to the provider. Session revocation stops refreshes; finalize immediate access-token invalidation versus short expiry before implementation.

Authentication is required for the application routes below unless explicitly noted.

| Method and route | Purpose and access |
| --- | --- |
| `GET /me` | Current user's profile and preferences |
| `PATCH /me` | Update own allowed profile fields, including preferredLocale as en or it |
| `DELETE /me` | Request account deletion; process hosted sessions and memberships safely |
| `GET /locations/search?q=...` | Rate-limited global address lookup for host venue entry; not an alternative discovery origin |
| `POST /venues` | Create an owned venue with explicit visibility |
| `GET /venues/mine` | List owned venues; exact details only for their personal owner |
| `PATCH /venues/:id` | Update an owned venue revision without silently relocating existing session snapshots |
| `GET /sessions` | Nearby published sessions with public fields and availability |
| `POST /sessions` | Create a draft with hostParticipates defaulting to true; create an approved host membership only when the host plays, in the same transaction |
| `GET /sessions/:id` | Public detail for published sessions; drafts only for their host |
| `PATCH /sessions/:id` | Host edits all business fields, including hostParticipates; require expected version and a valid resulting capacity |
| `POST /sessions/:id/publish` | Host validates and publishes a draft |
| `POST /sessions/:id/cancel` | Host cancels an active session; retries return its cancelled state |
| `GET /sessions/:id/location` | Exact location for host or approved participants in an active session |
| `POST /sessions/:id/requests` | Create or return own participation request |
| `GET /sessions/:id/participants` | Host sees requests; approved players see limited confirmed profiles |
| `PATCH /sessions/:id/requests/:requestId` | Host sets a pending request to approved or rejected |
| `DELETE /sessions/:id/participation` | Withdraw own membership idempotently |
| `GET /me/sessions?role=...&status=...` | Hosted, requested, or confirmed sessions, including cancelled history |
| `GET /me/notifications` | Initial or explicit reconciliation inbox read, with cursor pagination; not a polling endpoint |
| `PATCH /me/notifications/:id` | Mark own notification read idempotently |
| `PUT /me/push-devices/:installationId` | Register or update the current authenticated installation's push token and preferences |
| `DELETE /me/push-devices/:installationId` | Revoke the current user's installation token association |
| `POST /reports` | Report a session or player for operator review |
| `GET /health/live` and `GET /health/ready` | Public minimal service status; detailed diagnostics restricted |

Define a minimal operator report review and session-disable mechanism before public launch. Implement it as an operator command initially or a role-protected API; a full admin application can follow later.

## Discovery request example

```http
GET /v1/sessions?latitude=45.4642&longitude=9.1900&radiusKm=10&startsAfter=2026-12-01T00%3A00%3A00Z&startsBefore=2026-12-08T00%3A00%3A00Z&format=commander&minAvailableSeats=1&limit=20&offset=0
Authorization: Bearer <access-token>
```

Return session ID, title, format, UTC start/end, timezone, city, capacity, confirmed count, available seats, and version. Discovery exposes no venue name, exact or approximate venue coordinates, map point, address, or precise distance. Unapproved detail uses the same city-only disclosure. The protected location endpoint returns the exact address and point for the host or an active approved participant to render the detail map.

## Contract rules

- Validate DTOs with a global whitelist and reject unknown fields; derive actor and ownership IDs from the authenticated user.
- Return UTC ISO 8601 timestamps and accept explicit offsets or UTC. Resolve daylight saving ambiguity in the creation form before submission.
- Accept valid host venue coordinates worldwide. Discovery origin is the current device position; do not add a country restriction or manual destination mode. City-based geographic filtering and ranking never use secret exact session coordinates.
- Keep interface locale, session play language, and venue timezone independent. Validate preferredLocale against en and it. Stable error codes let the app localize signup/login and recovery errors without exposing credentials or changing the response schema.
- Return consistent errors with `code`, `message`, optional `fieldErrors`, and a request ID. Use `400` for invalid input, `401` for missing or invalid authentication, `403` for forbidden visible operations, `404` for missing or hidden resources, `409` for conflicting state, and `429` for throttling.
- Make publish, cancel, approve, and withdraw safe to retry. Use a per-user idempotency key on session creation to avoid duplicate sessions when a connection fails after saving.
- Return `Cache-Control: no-store` on protected location responses. Do not put private coordinates in discovery payloads, analytics, or push notification text.
- Push events carry a small complete safe UI projection, event IDs, and monotonic aggregate versions, within the provider's payload limit. The client deduplicates and applies newer events directly without a read on each push; it reconciles on app entry or explicit refresh. Protected location is fetched only for an authorized detail view and is cleared locally on revocation events.
- Generate and review OpenAPI examples for success, full tables, expired sessions, and authorization failures before integrating the mobile client.
