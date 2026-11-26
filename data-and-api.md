# Data model and API plan

Represent a scheduled table as a session and keep participation separate so requests, confirmed seats, and cancellations have explicit rules. The API must enforce the same rules for every client.

## Main entities

| Entity | Principal fields | Constraints |
| --- | --- | --- |
| User | UUID, display name, preferredLocale, adult declaration/terms/policy version/timestamps, onboarding, deletion and moderation state | No login username; name 2-40 characters without uniqueness; en/it preference; completed 18+ onboarding required for product access |
| City | UUID, unique provider locality ID, name, country code, canonical discovery centroid | Shared city-level search point; exact session coordinates never drive pre-approval discovery |
| AuthIdentity | UUID, user ID, provider, issuer/subject, encrypted revocation credential when required | Unique issuer/subject and user/provider; explicit fresh linking; no provider email/name merging |
| AuthSession | UUID, user ID, originating identity, installation ID, created/last-used/absolute expiry, revoked at | Multi-device; 30-day inactivity and 90-day absolute limit; every protected request checks current account/session state |
| RefreshToken | UUID, auth session ID, token hash, rotation family, expires at, consumed at, revoked at | Track rotation and detect reuse; invalidate affected session on confirmed reuse |
| Venue | UUID, personal owner ID, name, city ID, timezone, accessibility notes, category | Category public_space/private_home is descriptive only; exact location remains protected for both |
| VenueLocation | Venue ID, revision, exact address and point | Owner-only saved location, independently retained from session snapshots; no public coordinate access |
| SessionLocation | Session ID, exact address, exact point, city ID, venue revision | Protected snapshot for every venue type; edits do not relocate unrelated sessions |
| Session | UUID, host ID, hostParticipates, nullable venue ID, title, format, description, playLanguage, playStyleNotes, accessibilityNotes, starts at, ends at, timezone, capacity, lifecycle, version, completionGeneration, timestamps | hostParticipates defaults to true; end after start; accepted capacity 2 to 16; host owns the session |
| Participation | UUID, session ID, user ID, status, requested at, decided at, decided by | Unique session and user; statuses pending, approved, rejected, withdrawn |
| Format | Stable code, display label keys, enabled flag | Extensible catalogue; initial seeded values are the sole open choice |
| AuthTransaction | ID, provider/environment, nonce/challenge hashes, action/actor, expiry/consumption | Single use, ten-minute lifetime; browser completion code 60 seconds |
| CommandOperation | UUID, actor/auth session, action and target resource ID, UUIDv7 idempotency key/keyed digest, encrypted input, state/revision, result/error code, prepared/accepted/deadline/terminal times | Two-minute execution deadline, atomic terminal outcome/event, seven-day minimal receipt/result; private input maximum 24 hours |
| UserSyncCursor | User ID, monotonic recipient revision | Separate row from User; transactional resource projections and snapshot watermark |
| UserBlock | Owner user ID, blocked user ID, created at | Unique directed pair, no self-block; either direction forbids the interactions defined in product scope |
| DeletionRequest | UUID, user ID, requested at, synchronous effects committed at, provider/purge progress | Resume physical cleanup by request ID; never defer game cancellation or seat withdrawal |
| DeletionLedger | Deleted internal UUID, request/purge timestamps | Minimal restore suppression record; retain 35 days after purge and export encrypted for recovery |
| Report | UUID, reporter ID, session ID or user ID, category, details, state, timestamps | Authenticated creation; only operators can inspect reports |
| Notification | UUID, recipient ID/inbox revision, event ID, optional session ID, aggregate ID/version, type, safe payload, created at, read at | Unique recipient and event; no precise location or credentials in the payload |
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
| Player withdraws or leaves | Pending or approved to withdrawn | Own non-host membership while published and before current end; after-start withdrawal does not reopen joining |
| Host changes playing choice | Approved to withdrawn, or absent/withdrawn to approved | Session owner edits hostParticipates under the session lock; opting in must fit resulting capacity |

A cancelled session retains participation records for history, but allows no further request or approval and grants no private address access. Completion or reaching the current end time also ends private address access, even if the scheduler is late. Cancelled/completed history is terminal; duplicate into a new draft for another game. Only end/lifecycle changes advance completionGeneration, independently of aggregate version. Session status is checked alongside participation status on every protected operation.

## Prevent overbooking

1. Begin a database transaction and lock the session row with `SELECT ... FOR UPDATE`.
2. Recheck the host's authority, lifecycle, current server time, and the membership's current status.
3. Count approved participation inside the transaction. If approving a new member would exceed capacity, return `409 TABLE_FULL`.
4. Update the membership and commit. A retry for an already approved membership returns the existing success.

For business mutations lock the actor User row first, then its command/receipt if applicable, then affected session rows in UUID order. Every operation affecting capacity or eligibility acquires the session lock before membership locks: approval, withdrawal, host participation edits, capacity or schedule edits, cancellation, and completion. Acquire membership locks after session locks, then separate recipient UserSyncCursor rows in UUID order; never acquire actor/command/session locks after cursor locks. Internal expiry-only work locks its command before cursors and changes no business resource. This serializes competing actions without relying on a stale mobile count.

## Immediate security transactions

Account deletion/suspension, session cancellation, own withdrawal and blocking must not wait for a queue or push. Under the current actor's User lock (the target account for an operator suspension), validate the dedicated safety limit and current authorization; acquire own command receipts and all affected session/membership locks in the order above. Discover the actor's complete active host/participation set after acquiring that User lock; normal creation/request/host-opt-in paths take the same actor lock, so no new own membership can appear outside the enumeration. Session locks serialize concurrent approvals/edits. The active-resource caps below bound this work.

Deletion/suspension marks the account unavailable and revokes auth/push associations, cancels its draft/published games with reason HOST_UNAVAILABLE and advances their aggregate/completion generations, and withdraws its pending/approved memberships in other active games. Record recipient notifications/projections, revisions, safe outcomes and the DeletionRequest/ledger atomically before returning success. Confirmation counts use resulting approved rows; no cleanup delay keeps a seat occupied. A concurrent action validates host, candidate and account availability under its session lock and fails if deletion won. Cancelled/withdrawn/private-location guards apply to subsequent reads immediately. Retain earlier terminal history only as redacted history under retention. Suspension cannot be reversed before its cancellation cleanup is materialized, and reactivation never reopens old games.

Withdrawal/cancellation and safety blocking use the same immediate lock/update/event discipline. Before taking session locks, withdrawal locks the actor's nonterminal earlier request commands for that target and marks them rejected with OPERATION_SUPERSEDED, recording their outcomes in the same transaction. This also applies when no membership has materialized yet; a delayed accepted request cannot recreate participation after withdrawal acknowledgement. An execution already holding the actor lock finishes first and is then withdrawn normally. Apply [product block semantics](./product-scope.md#account-deletion-and-safety) to the blocker's complete active set after locking its User row, rechecking the target's current membership inside each session lock. New request/approval guards reject a block relation between candidate and host/any approved player. Blocking preserves other seats and emits generic withdrawal/invalidation projections; do not expose the blocking party to the other user. Unblock removes only the owned directed relation and never recreates memberships. Other-party deactivation/block races are tested over independent database connections.

After commit, enqueue delivery and physical provider/purge work using recorded IDs. On handoff failure retain the committed safety effect and authoritative inbox; targeted operator replay handles delivery, and retention maintenance resumes the DeletionRequest. Never report successful deletion/cancellation/block if the database transaction failed. A connection lost after commit repeats the same idempotency key where authentication remains available; a deleted/revoked caller instead clears local state and cannot retrieve another user's data. These safety endpoints are not rejected by ordinary write/geocoding quota buckets.

Protected location reads require an active caller/account, available host, published session with current end in the future, eligible membership and no host/caller block relation. Recheck after account disable even when an old JWT or event says approved. Discovery and ordinary detail exclude unavailable hosts and prohibited block relations; historical own records are redacted instead of exposing former addresses.

## Validation and admission limits

These are accepted pilot limits, enforced transactionally across serverless instances. They are configurable operational settings in version control, never client authority. Apply the stricter user/IP limit; shared IP buckets use privacy-preserving keyed hashes and expire within 24 hours. Existing auth-specific limits remain in [Authentication](./authentication.md).

| Resource or operation | Limit |
| --- | --- |
| Owned saved venues / unexpired drafts | 20 venues / 10 drafts per user |
| Published future/ongoing hosted games | 5 per user; safety cancellation does not consume a creation allowance |
| Own active memberships | 20 pending/approved memberships, including playing-host seats; pending-to-approved changes do not add another membership |
| Pending requests per session | 50; reject excess with `409 REQUEST_LIMIT_REACHED`; no pending seat reservation |
| Nonterminal interactive commands | 3 per user within their execution deadline; receipts past deadline do not occupy the active-command cap |
| Owned block relations | 500; at capacity let the user remove an old block or report, never silently unblock |
| New session creation | 5/hour per user, 20/hour per IP, in addition to the active/draft caps |
| Other ordinary business writes | 30/minute per user, 60/minute per IP; at most 20 ordinary field edits per session per 24 hours, excluding safety actions |
| Safety block/unblock, cancellation, withdrawal | Separate 10/minute per user and 30/minute per IP bucket; same-key retries return the receipt without consuming another action |
| Deletion and logout | Separate existing auth/security buckets; ordinary business/provider quotas cannot prevent these actions |
| Protected ordinary reads / snapshot | 60/minute per user, 120/minute per IP / snapshot 6/minute per user, 20/minute per IP |
| Discovery | 20/minute per user, 60/minute per IP; radius 1-100 km, page maximum 50, date window maximum 90 days |
| Host geocoding | 10/minute per user, 30/minute per IP, global upstream maximum 5/second within provider quota; debounce 500 ms and query length 3-200 |
| Reports | 5/hour per user, 20/hour per IP; coalesce duplicate open reports by reporter/target/category |
| Public business DTO body | Maximum 16 KiB; dedicated auth/provider/Workflow handlers have explicitly tested separate SDK-appropriate bounds |
| Display name / session title / venue name | 2-40 / 3-80 / 2-80 Unicode code points after trim; reject controls |
| Session description / play-style notes / accessibility notes | Maximum 2,000 / 500 / 500 code points; plain text |
| Session playLanguage | Valid BCP 47 tag, maximum 35 ASCII characters, independent of en/it interface locale |
| Exact address / private report details | Maximum 500 / 2,000 code points; never public/push/log payloads |
| Publication/schedule edit | Start no more than 90 days ahead; positive duration at most 24 hours. Published end must remain after server now; an ongoing game's start can remain past |

Use `429` with Retry-After for frequency limits, `409` for resource/command-cap conflicts and `400` for DTO shape/field errors; oversized public bodies return `413 PAYLOAD_TOO_LARGE`. Apply actor/resource caps and key admission before starting new work; recheck membership/capacity/block constraints inside the transaction. Geocoding/map provider exhaustion produces explicit unavailability, with no paid fallback. Authorized address text can remain visible when tiles are unavailable; an unresolved canonical city blocks publication. Stable client validation mirrors server rules, while server checks remain authoritative.

Public names/session text pass a versioned deterministic moderation filter before save/publication, returning `422 CONTENT_NOT_ALLOWED` on prohibited content without committing it. Private report evidence/address fields are not treated as public content and are never echoed in public errors. Accept playLanguage as a valid BCP 47 tag independently of interface locale. The session DTOs include this field and the bounded playStyleNotes/accessibilityNotes above.

## REST endpoints

All routes below use `/v1`, except health. Contracts use the accepted [authentication implementation](./authentication.md); route-level implementation is still required.

| Method and route | Purpose and access |
| --- | --- |
| `POST /auth/:provider/start` | Create single-use transaction/nonce and app challenge; only Google or Apple, fixed environment and action |
| `POST /auth/apple/callback` | Android browser form-post return; validate state/code/nonce and redirect only an opaque 60-second completion code |
| `POST /auth/complete` | Redeem browser completion code with app-held verifier; no credentials in app-return URL |
| `POST /auth/apple/notifications` | Verified Apple server-to-server revocation/account-event callback; no application bearer credential, mandatory signed provider validation |
| `POST /auth/:provider/exchange` | Verify supported native SDK credentials with transaction proof; resolve/create the application user and issue the app session |
| `POST /auth/refresh` | Authenticate with a valid refresh token, rotate it, and issue a new short-lived access token |
| `POST /auth/logout` | Revoke the current authentication session using a valid session credential; repeated logout is safe |
| `POST /auth/logout-all` | Authenticated operation revoking all of the user's authentication sessions |
| `GET /me/auth-sessions` | List own devices/sessions with redacted metadata |
| `DELETE /me/auth-sessions/:id` | Revoke one owned session immediately |
| `GET /me/identities` | List linked provider labels without credentials |
| `POST /me/identities/:provider/link` | Complete explicit linking with fresh proof of existing and new provider identities |
| `DELETE /me/identities/:provider` | Fresh proof of remaining provider; reject last-provider removal and revoke all sessions |

The app receives the API's access token, not a provider token for general API authorization. Native exchange is used for Google on both platforms and Apple on iOS; browser completion is used for Apple Android. First sign-in creates an onboarding account under issuer/subject uniqueness; settings link/unlink only under the accepted fresh-proof rules. No local registration, password reset, or recovery code endpoint exists. Provider account recovery belongs to the provider. Session revocation stops refresh and access immediately through the current session/account check on protected requests; access JWTs last 15 minutes and refresh sessions have 30-day inactivity/90-day absolute limits.

Authentication and completed onboarding are required for product routes; profile/onboarding, identity/session settings and own account deletion remain available to incomplete accounts. Provider callbacks use their own signed/transaction validation, never an application-auth bypass. Health is the only public diagnostic route.

| Method and route | Purpose and access |
| --- | --- |
| `GET /me` | Current user's profile and preferences |
| `PATCH /me` | Update own profile/locale and complete versioned adult declaration/policy acknowledgement; user cannot grant roles or reset recorded age state to bypass onboarding |
| `DELETE /me` | Fresh proof and atomic account/session disable, game cancellation, own withdrawal, revisions/events and deletion ledger; provider/purge jobs run afterwards |
| `POST /locations/search` | Rate-limited host address lookup with input in the body and logging disabled; not an alternative discovery origin |
| `POST /venues` | Create an owned venue with public_space/private_home category and owner-only precise location |
| `GET /venues/mine` | List owned venues; exact details only for their personal owner |
| `PATCH /venues/:id` | Update an owned venue revision without silently relocating existing session snapshots |
| `DELETE /venues/:id` | Delete the owned template/location; session venue references use ON DELETE SET NULL and protected snapshots remain independent. Confirm that active games keep their snapshot until cancelled/relocated/expired |
| `GET /me/snapshot` | One repeatable-read snapshot, recipient watermark and complete bounded active set, newest inbox/outcomes; entry/re-entry or explicit refresh only |
| `GET /me/blocks` | Own block list with paginated redacted identity labels |
| `PUT /me/blocks/:userId` | Immediate owned block plus shared-game safety effects; only a visible/previously shared user target, never self |
| `DELETE /me/blocks/:userId` | Remove only own directed block, without membership restoration |
| `GET /formats` | Enabled format catalogue with stable codes and localized labels; seed after initial formats decision |
| `POST /discovery/search` | Nearby city-only sessions; current device position in the body, never the URL/query string |
| `GET /me/operations/:id` | Own command state/revision/deadline and safe terminal result/error; bounded lock/expiry arbitration; explicit recovery only |
| `POST /sessions` | Create a draft with hostParticipates defaulting to true; create an approved host membership only when the host plays, in the same transaction |
| `GET /sessions/:id` | City-only detail for published sessions, still requiring authentication; drafts only for host; historical detail only for host/own participants |
| `PATCH /sessions/:id` | Host edits all business fields, including hostParticipates; require expected version and a valid resulting capacity |
| `POST /sessions/:id/publish` | Host validates and publishes a draft |
| `POST /sessions/:id/cancel` | Host cancels an active session; retries return its cancelled state |
| `GET /sessions/:id/location` | Exact location for host or approved participants in an active session |
| `POST /sessions/:id/requests` | Create/return own request with expectedVersion; a synchronous withdrawal fences earlier pending request commands |
| `GET /sessions/:id/participants` | Host sees requests; approved players see limited confirmed profiles |
| `PATCH /sessions/:id/requests/:requestId` | Host sets a pending request to approved or rejected |
| `DELETE /sessions/:id/participation` | Immediate idempotent own withdrawal until end, including after start; never reopens joining by itself |
| `GET /me/sessions?role=...&status=...` | Hosted, requested, or confirmed sessions, including cancelled history |
| `GET /me/notifications` | Initial or explicit reconciliation inbox read, with cursor pagination; not a polling endpoint |
| `PATCH /me/notifications/:id` | Synchronous owned read acknowledgement with recipient revision; update that inbox resource, never create another unread command-outcome notification |
| `PUT /me/push-devices/:installationId` | Register or update the current authenticated installation's push token and preferences |
| `DELETE /me/push-devices/:installationId` | Revoke the current user's installation token association |
| `POST /reports` | Report a session or player for operator review |
| `GET /health/live` and `GET /health/ready` | Public minimal service status; detailed diagnostics restricted |

Use the restricted operator CLI for report review, account disable and unsafe-session cancellation, with audited actor/reason/time. Do not expose a public admin role assignment or build an admin UI in the MVP.

## Discovery request example

```http
POST /v1/discovery/search
Authorization: Bearer <access-token>
Content-Type: application/json

{
  "latitude": 45.4642,
  "longitude": 9.1900,
  "radiusKm": 10,
  "startsAfter": "2026-12-01T00:00:00Z",
  "startsBefore": "2026-12-08T00:00:00Z",
  "minAvailableSeats": 1,
  "limit": 20,
  "offset": 0
}
```

Return session ID, title, format, playLanguage, safe host ID/display name, UTC start/end, timezone, city, capacity, confirmed count, available seats and version. Discovery exposes no venue name, exact or approximate venue coordinates, map point, address, or precise distance. Unapproved detail uses the same city-only disclosure. The protected location endpoint returns the exact address and point for the host or an active approved participant to render the detail map.

## Contract rules

- Validate DTOs with a global whitelist and reject unknown fields; derive actor and ownership IDs from the authenticated user.
- Return UTC ISO 8601 timestamps and accept explicit offsets or UTC. Resolve daylight saving ambiguity in the creation form before submission.
- Keep current-position discovery coordinates in the POST body with request-body logging disabled, not in a URL captured by platform access logs. Return no-store on authenticated search. Accept valid host venue coordinates worldwide. Discovery origin is the current device position; do not add a country restriction or manual destination mode. City-based geographic filtering and ranking never use secret exact session coordinates.
- Keep interface locale, session play language, and venue timezone independent. Validate preferredLocale against en and it. Stable error codes let the app localize signup/login and recovery errors without exposing credentials or changing the response schema.
- Return consistent errors with `code`, `message`, optional `fieldErrors`, and a request ID. Use `400` for invalid input, `401` for missing or invalid authentication, `403` for forbidden visible operations, `404` for missing or hidden resources, `409` for conflicting state, and `429` for throttling; use `413` for oversized public bodies and `422 CONTENT_NOT_ALLOWED` for rejected public content.
- Require expectedVersion for ordinary session-targeting commands, including request and approval, to reject intent based on stale details. Safety operations apply to the latest state and can revoke despite stale UI. Require per-user idempotency keys for creation and notification-producing mutations. Use the durable command contract: committed success/domain error, or 202/pending with an operation ID after durable acceptance; no client polling. Retrying the same key/input returns the same operation/result. Never show a confirmed seat for a pending command. Keys, outcomes and expired-retry behavior follow [Delivery and recovery](./delivery-and-recovery.md#idempotency-window).
- Return `Cache-Control: no-store` on protected location responses. Do not put private coordinates in discovery payloads, analytics, or push notification text.
- Push events carry a small complete safe UI projection, event IDs, and monotonic aggregate versions, within the provider's payload limit. The client deduplicates and applies newer events directly without a read on each push; it reconciles on app entry or explicit refresh. Protected location is fetched only for an authorized detail view and is cleared locally on revocation events.
- Generate and review OpenAPI examples for success, full tables, expired sessions, and authorization failures before integrating the mobile client.
