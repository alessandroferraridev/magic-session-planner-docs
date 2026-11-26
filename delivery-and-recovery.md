# Durable delivery and recovery

Use Vercel Workflow SDK (`workflow` and `@workflow/nest`) with managed Vercel execution/queues and Expo push. This is the implementation contract; runnable jobs and staging evidence remain roadmap work. No notification-table poller, always-on worker, Redis service or websocket channel is required.

## Durable command boundary

A business commit followed by untracked sending can lose every push. Notification-producing business mutations use the following protocol:

1. Authenticate/validate the request and its [idempotency key](#idempotency-window). Store a CommandOperation containing actor/auth-session IDs, expected aggregate version, keyed request digest, encrypted input, operation revision, timestamps and an execution deadline two minutes after first validated preparation; acceptedAt is set only after durable start. Preparation changes no game state. Retries keep the original deadline and operation ID.
2. Start a durable workflow with only that operation ID. Acknowledge acceptance only after durable start succeeds. A crash before start leaves an unaccepted prepared command that a same-key request can resume; no periodic scan dispatches prepared commands. Duplicate workflow starts are harmless under database idempotency. Persist the accepted run ID/acceptedAt with a conditional prepared-to-queued transition; if a worker already committed an outcome, acknowledgement metadata must not regress its terminal state or reset its deadline.
3. Lock the actor User row and then the command row, and return a stored terminal outcome if one exists. Otherwise recheck current actor/account/session authorization, deadline, expected version, blocks, quotas and domain rules. Lock session rows before memberships. Apply business effects, aggregate version, recipient revisions, notifications, delivery events and the command's terminal result in one transaction. Erase private command input. For a domain rejection or elapsed deadline, commit only the terminal outcome and its actor event; do not mutate the game.
4. Durable steps dispatch committed safe projections and schedule lifecycle work. A crash after commit resumes dispatch without repeating business effects. Workflow arguments/results contain only IDs and redacted outcomes, never addresses, credentials or command bodies. A separate deadline step arbitrates expiry under the same command lock; expiry and success cannot both win.
5. Wait at most ten seconds for the outcome. Return normal success/domain error if available; otherwise return `202` with `operationId`, `status`, `revision`, `acceptedAt`, `executionDeadlineAt` and `serverTime`. Acceptance never confirms a seat. The client does not poll.

Use this protocol for creation, publishing, ordinary edits and participation changes. Immediate deletion, suspension, withdrawal (including superseding earlier queued requests), cancellation and blocking use the bounded synchronous safety transaction in [Data and API](./data-and-api.md#immediate-security-transactions); push availability cannot delay revocation. Their persisted notifications remain authoritative and dispatch is attempted after commit. If that handoff fails, the security effect stays committed and targeted operator replay uses the recorded event IDs; no notification-table polling repairs it. Physical deletion has a resumable DeletionRequest, separate from notification dispatch. Read acknowledgements, push registration and UI preferences are synchronous idempotent control updates: return their versioned projections and send best-effort control events to other owned devices, but create no CommandOperation or new unread operation notification.

The [NestJS Workflow integration](https://workflow-sdk.dev/docs/getting-started/nestjs) is currently beta. Pin its SWC integration and Vercel build output, including the generated consumer. Workflow steps do not inherit Nest dependency injection: initialize bounded adapters inside steps and call shared business logic. Prove actual crash recovery and included-quota behavior before pilot. See [Workflow limits](https://vercel.com/docs/workflows/pricing) and [run access](https://workflow-sdk.dev/docs/api-reference/workflow-api/get-run).

## Command outcomes and pending UI

| Server state | Meaning | Observable result |
| --- | --- | --- |
| prepared | No accepted durable run and no game effect | Original same-key request can retry start; not reported as accepted |
| queued / running | Durable acceptance, outcome not committed | `202`, original deadline and Check result action |
| succeeded | Business effect and outcome committed atomically | Safe result/resource identifiers and current authorized projections |
| rejected | Domain or authorization condition failed | Stable error code, equivalent HTTP status, safe current state where visible; no game effect |
| expired | Deadline passed before execution won the command lock | `409 OPERATION_EXPIRED`; no game effect; intentional retry needs a new command/key |

Keep command outcome separate from push-delivery status. Every terminal transition creates an actor `operation.completed` event/notification, including rejections and expiry, with operation revision, outcome and safe error/result IDs. Events persist in-app even if the original response was synchronous. A delivery failure cannot turn a successful seat into a failed command. Invalid/disabled actors receive no private projection or push; an unauthorized client clears its state and is denied the result.

After `202`, display In progress and Check result. With push denied, explain that explicit checking is available. At the returned deadline a local UI timer changes the label to Outcome unknown if no terminal result was received; it performs no network or database request and does not claim failure. Success may already have committed and its push been lost. User checks, entry/re-entry snapshots and terminal pushes resolve this state. No automatic new-key retry or timer-based status read is allowed.

`GET /me/operations/:id` reads the current outcome with a bounded command-row lock, so a commit in flight cannot be misreported as expired. If queued work is past its deadline, it can atomically materialize expiry and its terminal event; this is explicit-request housekeeping, not a polling loop. If the row is busy, return a retryable unavailable response without inventing an outcome. Missing/foreign resources remain hidden. Preserve terminal results independently of whether their push was received.

## Idempotency window

- Business command keys are client-generated UUIDv7 values in `Idempotency-Key`. Store a unique `(userId, key)` receipt for every prepared command and the keyed HMAC-SHA-256 digest of route/action/normalized input; keep the digest key outside PostgreSQL. Auth-token refresh uses its own rotation protocol, not this contract.
- For an existing receipt inside its seven-day logical replay window, check actor authorization and return the original operation/result before applying key-age admission rules. A receipt beyond that window returns IDEMPOTENCY_KEY_EXPIRED without another effect, even if awaiting physical purge. Changed action/input with the same key returns `409 IDEMPOTENCY_CONFLICT`, without repeating effects. Input deletion does not remove its digest.
- For an unknown key, accept only a UUID timestamp no more than 24 hours old and no more than five minutes ahead of server time. Reject old keys with `409 IDEMPOTENCY_KEY_EXPIRED`; invalid/future keys return `400 INVALID_IDEMPOTENCY_KEY`. Device clock errors show a correction/retry action, never silently mint another command after an ambiguous result.
- Keep minimal receipt/outcome/digest usable for seven days after terminal completion, then physically purge on the next daily run within eight days; expired unaccepted preparation also creates a receipt outcome. Inputs expire within 24 hours at most and normally disappear on the two-minute outcome. Never purge a nonterminal receipt and then re-admit its key. Once a logically expired receipt is physically gone, the original UUID timestamp is already too old to create new effects. After receipt/result purge, GET by operation ID returns `404 OPERATION_NOT_FOUND`; POST with its original old key returns `409 IDEMPOTENCY_KEY_EXPIRED`. Never infer ownership from an untrusted operation ID.
- Serialize duplicate requests under the receipt/command uniqueness and row lock. Ambiguous connections retry the original key; new keys represent new explicit user intent. Account deletion can remove own receipts under its earlier purge policy because that old user UUID can never authenticate again.

UUIDv7 timestamp semantics are specified in [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html#name-uuid-version-7). This finite retention contract prevents an old retry from recreating a session after the result has been cleaned up.

## Notification contract

Every affected recipient has a separate UserSyncCursor. Allocate monotonic recipient revisions in the same transaction as business state and notification changes; lock recipient cursor rows in UUID order after session/membership locks. Session aggregate version still governs domain edits; recipient revision orders personalized cache state, command outcomes and inbox changes.

Push envelopes contain `eventId`, recipient scope, recipient revision and small complete resource projections keyed by resource type/ID: session summary/own membership, notification, command outcome or invalidation tombstone. Include aggregate version where relevant. A tombstone removes access/state and contains no former address. Keep the full serialized envelope below 4 KiB; send a small invalidation notice for descriptive fields not represented, with detail fetched only on explicit navigation. No exact location, venue name, credential, raw report, prohibited content or long user text enters push payloads/workflow history.

Use the same merge rules for mutation responses, command outcomes and push envelopes. Deduplicate event IDs, compare revisions per resource key and ignore older/equal projections. A delayed event for a different key may still apply; never drop it solely because another resource advanced the global high-water mark. Retain in-memory tombstones until a new authoritative snapshot establishes a newer baseline. Mark revision gaps as needing reconciliation but do not fetch for every push.

Revocation, withdrawal, relocation, cancellation, block, moderation, logout and deletion clear precise-location state immediately when observed. API authorization always rechecks the current committed state independently of push delivery; an address already seen cannot be made unknown and a lost push cannot guarantee instant erasure from an offline device. Keep map data in memory only, clear it on re-entry/expiry/logout, and load it again only for an explicitly opened authorized detail.

## Snapshot and event merge

`GET /me/snapshot` is the single entry/re-entry reconciliation request. In one PostgreSQL repeatable-read transaction return authenticated user scope, `serverTime`, recipient watermark W, the complete bounded set of active hosted/pending/approved game summaries, current safety/visibility tombstones needed by that set, up to 50 newest inbox entries plus unread count/cursor, and up to 20 recent command outcomes plus all nonterminal operations. Include per-resource revisions and pagination cursors capped at W; historical lists and later pages load only on explicit navigation/scroll. Public invalidations reveal no blocked-party identity.

1. Capture the current user ID and a client auth-generation value. Register the foreground listener before requesting the snapshot; buffer incoming events and mutation responses during the request, for at most 200 envelopes. On overflow discard the excess and mark refresh required, without adding another automatic read.
2. Apply the response only if user/auth-generation/request-generation still match. Replace the complete active set at W, clear obsolete precise data and historical query pages, and install the inbox/operation page. Remove absent active keys; do not merge missing items back from old caches.
3. Replay buffered projections newer than W using per-resource revisions/tombstones. Events at or below W are covered by the authoritative baseline and cannot resurrect removed data. Listener events arriving later use the same rule. A snapshot begun before a newer push therefore cannot overwrite that newer state.
4. On failure keep existing public summaries visibly stale, clear sensitive maps, replay buffered events safely and show Retry. Logout/account switch clears the buffer and increments auth-generation, so late replies cannot populate another account. Overlapping snapshot responses apply only to the newest request generation.
5. A gap/overflow displays an explicit Refresh action. At most one automatic snapshot runs per launch/login/foreground activation; deduplicate overlapping triggers. No push-triggered snapshot, timed HTTP request, background refresh job or operation polling is permitted.

An inbox read-state change also advances recipient revision and sends a safe inbox projection to other owned installations; delayed create events cannot mark a read notification unread. Background/killed apps may not run a listener; APNs/FCM/Expo delivery is best effort, and entry/re-entry or user refresh is the recovery mechanism.

## Bounded push execution

Use `expo-server-sdk`, server-held push credentials and environment-specific device associations. Batch within Expo limits. Attempt at most five sends over a 24-hour delivery deadline with jitter around one, five, fifteen and sixty minutes and `Retry-After`; never retry permanent errors. Sleep schedules one known-ticket receipt check around 15 minutes after issue and at most one further check for pending receipts. Disable `DeviceNotRegistered` tokens. Unknown final delivery is diagnostic state; the inbox/outcome remains authoritative. Receipt checks never scan the notification table. See [Expo sends and receipts](https://docs.expo.dev/push-notifications/sending-notifications/).

Only the current user owns an installation association; rebind/remove atomically on account switch/logout and recheck association before sending. Push preference follows OS permission and in-app settings. Permission denial never blocks participation, terminal-outcome reads or safety actions.

## Scheduled lifecycle and maintenance

Publishing/editing end time schedules completion by session ID, completionGeneration and expected end. Increment this generation only for end/lifecycle changes; other aggregate revisions cannot suppress completion. Jobs recheck generation/end/status. Authorization, joining cutoffs and active edits use current server time even if jobs are late. Cancelled/completed history is terminal; another game starts with a new draft.

A protected daily 03:00 UTC Vercel invocation starts bounded retention/expiry batches with durable continuation. It also resumes known DeletionRequests whose physical-purge handoff failed; game cancellation and access revocation already committed synchronously. It never scans notifications for dispatch. Apply [Operations](./operations.md#retention-schedule) deadlines and environment isolation.

## Acceptance evidence

Prove accepted-run/commit/dispatch crash boundaries, duplicate starts, terminal rejection/expiry delivery, deadline races, old-key retries after receipt purge, outcome-unknown UI, snapshot/push ordering, tombstone non-resurrection, inbox read ordering, user switches and buffer overflow without hidden requests. Also prove safety effects with the queue stopped, actual pinned consumer deployment and quota handling. The tests are specified in [Testing strategy](./testing-strategy.md#edge-case-regression-scenarios); these gates are implementation work, not a claim of executed application tests.
