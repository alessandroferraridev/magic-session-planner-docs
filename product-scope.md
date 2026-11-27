# Product scope

The first release helps individuals arrange an in-person Magic: The Gathering game anywhere in the world, with nearby discovery around the device's current location. A table represents one scheduled session with a fixed player capacity, hosted by a person.

## Audience and hosting

Personal-account Magic players aged 18+ worldwide. Require self-declaration and versioned policy acknowledgement, not birth dates or IDs; this is not verified age. Private homes are supported in the pilot/MVP with city-only discovery and approval-gated precise detail. The MVP is free to use and personal/non-commercial, with no store profiles, payments, advertisements, tournament brackets, paid features or child-directed flows.

Everyone who completes the required personal-profile onboarding can host. Venue categories describe public spaces or private homes; neither category discloses an exact location before approval. Provide in-app reporting and account deletion. Operator response duties and support contacts are defined in [Operations](./operations.md#moderation-and-support-operations).

## MVP requirements

- A host approves seat requests. Pending requests do not reserve seats.
- Anyone with a personal profile can host. Profiles represent individuals; store or organization accounts and business hosting features are outside scope.
- At creation, the host chooses whether to play or only provide the table. Playing is selected by default. A playing host occupies one seat; a host who only provides the table occupies none. A four-player session therefore begins with three or four available seats respectively.
- Authentication uses Google and Apple only. NestJS owns internal profiles and application sessions; there is no local username/password, email login, or local account-recovery system. Public display name is separate from provider identity.
- The app targets iOS and Android through Expo.
- Session hosting and discovery are available globally, without a launch-country or region restriction.
- Nearby discovery uses the current device position. The discovery list and unapproved session detail expose only the city; an approved participant sees the exact location and map in session detail. Hosts can inspect their own exact locations.
- In-app and push notifications are required in the MVP. Fetch once on app entry and update the UI from push payloads without timed polling or a new read for every notification.
- Hosts can edit all session business fields and change their own playing choice after creation. Capacity and ownership invariants still apply.
- The interface supports English and Italian. Language selection is independent of where a session is hosted.
- API and app development must include unit, integration, and E2E tests, implemented alongside their features from the initial scaffold. See the [testing strategy](./testing-strategy.md).

## Format coverage and scope boundaries

All formats are in scope for session planning from the first release: official entries, community/historical variants and a structured Custom path. [Formats and session configuration](./formats.md) owns the catalogue, media/preparation/layout differences and required host fields. Authentication and operations retain their dedicated specifications; no product choice remains open.

- Use versioned catalogue identities/profiles with aliases and immutable session rule snapshots. Unknown named formats use the moderated Custom/host-declared path; they do not require arbitrary global database codes. Support every family at launch rather than postponing non-Commander formats.
- The app uses foreground location only.
- Account access is required to browse and participate. Payments, chat, competitive matchmaking, deck management, ratings, and tournament brackets are outside the MVP.

## Global availability and interface languages

Hosts can create a session at any valid location worldwide. Players discover sessions around their current device position wherever they are. This does not introduce a manually selected search destination or a country restriction. Venue positioning during hosting is a separate action from choosing the discovery origin.

Provide English/Italian translations for navigation, social login and profile setup, validation, empty/error states, and provider recovery guidance. Session titles, descriptions, and venue names remain user content without automatic translation. A session's play language is separate from interface language.

Use the venue's IANA timezone to interpret and display session time, retaining UTC instants for storage. Choosing English or Italian does not change a session's location or timezone. An invited pilot is a validation exercise and does not define a geographic restriction for the product.

## Main user flows

### Find a table

1. Sign in and open nearby sessions.
2. Explain why location is useful, then request foreground permission when the player chooses to use their location.
3. Fetch the current foreground device position. If permission is denied, location services are off, or lookup fails, show settings and retry actions. Nearby discovery waits for a usable current position; personal upcoming and hosted session lists remain usable.
4. Show a list of future, published sessions with seats available. Default to a 10 km search radius, adjustable from 1 to 100 km, and a date window of the next seven days.
5. Show the format/variant, paper or device-play medium, intent, start time, city, total available human seats and host name. The location label is the city only, without venue name, address, map, or exact coordinates. Filter by date/radius, canonical format/family/modifier, medium/intent and player count; rank geographic results using city-level discovery data.
6. Open session details and request one seat. Clearly show pending, approved, rejected, and cancelled outcomes.

The list must handle loading, no results, stale results, location failure, and network failure. Once approval is confirmed, session detail loads the protected exact address and map. The list remains city-only even for approved players. The precise map is part of the MVP detail flow.

### Host a session

1. Choose a venue or create one, including its public area and location visibility.
2. Choose a searchable format recipe/profile, then its conditional deck/pool, preparation, material, layout/team/role and paper/device fields. Enter title, time/timezone, total human capacity, language, accessibility and play-style notes. Distinguish current/frozen rules and show any host-declared deviations. Choose whether to join as a player or only provide the table; the player option is selected by default and the available-seat preview reflects it.
3. Validate the draft and publish it. For private venues, show the host the exact address separately from the public area preview.
4. Review requests and approve players while capacity remains. The host sees confirmed players and pending requests.
5. Edit title, description, venue, location, format, time, capacity, and playing choice, or cancel the session. Existing confirmed players do not force cancellation and recreation. Persist a new session version and notify affected pending and approved players about material changes, with no exact address in notification payloads.

Capacity can be changed but cannot fall below the resulting confirmed player count. It counts people across all games, not decks, teammates or automated adversaries. Per-game team/pod sizes are a separate validated layout. A source/rules/layout change is material and cannot silently evict players; incompatible assignments are cleared and readiness updated. A playing host opting out frees one seat; opting in requires one available seat, or a valid capacity increase in the same edit. These changes are transactional. Publishing does not reserve a third-party venue; the individual host remains responsible for any venue booking.

### Manage participation

Players can see hosted sessions and their pending or confirmed sessions, withdraw a pending request, and leave a confirmed session at any time before it ends, including after the start. Leaving immediately frees a seat. Hosts keep management rights when they opt out of playing and can opt back in while a seat is available. Ownership transfer and arbitrary host removal of confirmed players are outside the MVP; safety blocking has the explicit membership effects below. Players retain reporting and withdrawal flows, and the operator can cancel unsafe sessions.

The app loads an initial notification and personal-session snapshot. Push listeners update local UI state directly using safe versioned event payloads for requests, decisions, cancellations, and material edits, without a backend read for every push. Opening protected precise detail remains an authorized user action. Reconcile once when entering or returning to the app, or on explicit refresh, to recover missed pushes. Denied push permission does not block participation or erase persisted notifications; immediate updates then wait for a one-shot refresh.

## Changes during a game

Hosts retain all business-field edits until the current end time. Validate the resulting end after start and after current server time; an already-started game may keep its past start. Terminal games cannot be reopened by editing dates. Preserve existing memberships after a material change, display an updated-details notice, and notify affected users; no automatic reapproval is required. Participants can withdraw until the end and immediately lose exact-location access. Leaving after the start does not reopen joining: ordinary requests and approvals still require a future start. If a host deliberately reschedules the start into the future, the normal future-session rules apply again.

## Account deletion and safety

After fresh verification, account deletion immediately revokes access, cancels every active hosted game and withdraws every active pending/approved membership in one transaction. Seats and protected-location access change before success is acknowledged, even when push delivery or physical purge is unavailable. Future reads cannot discover those hosted games or return their precise location. History is redacted and data purged under the retention policy; asynchronous jobs handle provider grant revocation, redaction and physical purge, not delayed cancellation. Deleting accounts cannot sign back in to reverse deletion. Transaction/locking details are in [Data and API](./data-and-api.md#immediate-security-transactions).

Users can report a session/profile and block another user from a visible host/participant profile or their block settings. Blocking forbids new host/player interaction whenever either direction is blocked, hides the other user's hosted games in discovery and denies their precise-location access. Apply these effects immediately to every shared active game:

- If either user hosts it, withdraw the other user's pending/approved participation.
- If both are ordinary players, withdraw the blocking user's participation; warn about this effect before confirmation.
- Reject a new request/approval when the candidate has a block relation with the host or any approved player. Two blocked pending applicants may exist; approving the first makes the other's approval ineligible.

Keep remaining players and the host's ownership unchanged. Use a generic unavailable/withdrawn result for the other party without naming who blocked whom. Existing shared history shows only the current user's redacted record. Unblocking restores neither membership nor location access; the normal request/approval flow applies. Blocking is a safety action, not a mechanism to disclose another user's block list. Users can always report known prior interactions, including after withdrawal or blocking.

Before posting, acknowledge versioned terms/community rules defining prohibited harassment, hate, threats, sexual content, impersonation and disclosure of someone else's private information. Server-side deterministic filters reject prohibited public names, titles, descriptions and notes; user text is plain text with no active HTML or links. Publish the rules in English/Italian and support private appeals/reports; automated filters do not guarantee detection in every language. Reports and sensitive location fields remain private. Detailed moderation execution belongs to [Operations](./operations.md#moderation-and-support-operations).

## Pending commands and recovery

A successful response confirms a result; `202` confirms only that an operation was accepted. Display an in-progress state with a Check result action, never a confirmed seat. Terminal command outcomes are delivered by versioned events. If push is unavailable or no outcome arrives by the returned execution deadline, show Outcome unknown and let the user perform one explicit status check. A local UI deadline changes text only and sends no request. Entry/re-entry snapshots recover pending/outcome state; there is no polling or automatic resubmission with a new key. The exact state machine and snapshot protocol are in [Delivery and recovery](./delivery-and-recovery.md).

## Demonstration experience

Offer Try demo on the signed-out screen in both languages. A permanent Demo / fictional data banner and synthetic host/player role switch let anyone, including store reviewers, exercise hosting, discovery, requests, approval, leaving, blocking, reports, maps and simulated notifications without a real local table or social credentials. Demonstration geography is explicitly fictional; real discovery still requires the current device position. Demo data and role changes never create real accounts, sessions or messages. Exit/reset clears the synthetic state. Production sign-in and native provider behavior remain separately verifiable; release packaging and review acceptance are defined in [Quality and release](./quality-and-release.md#store-review-access).

## Session rules

| Rule | MVP behavior |
| --- | --- |
| Lifecycle | Draft, published, cancelled, completed; cancelled/completed history is terminal. Duplicate into a new draft for another game |
| Full table | Derived from confirmed participation and capacity; not a separate lifecycle state |
| Availability | Published, starts in the future, and confirmed players below capacity |
| Join request | One request per player per session; every non-host player needs host approval |
| Host participation | Default true at creation; host can later opt out to free a seat or opt in when capacity allows |
| Host permissions | Ownership grants management independently of playing status; all business fields remain editable in draft/published active sessions, with valid time/capacity |
| Duplicate action | Retried requests must not create duplicate membership or allocate another seat |
| Completion | End-time authorization applies immediately; an idempotent scheduled job persists completion using a separate completion generation, unaffected by seat changes |
| Joining cutoff | Requests and approvals stop at the session start time |
| Location visibility | City only in discovery and before approval; exact address and map only for the host and active approved participants in active session detail |
| Time and layout | Store UTC instants and venue IANA timezone. Human capacity 1-16, with solo capacity only for an explicit solo/cooperative/custom profile. Publishing needs future time and structurally valid format setup, not already-full attendance |

## Required mobile screens

1. Sign in with Google or Apple, complete the 18+ declaration/policy acknowledgement and public display name on first access; settings include linked providers, sessions and logout controls.
2. Current location explanation, permission handling, settings recovery, and retry.
3. Nearby session list and filters.
4. Session detail and participation state, including protected precise location and map after approval.
5. Create and edit all session business fields, including a city-only preview and editable host participation.
6. Hosted sessions and request management.
7. My upcoming sessions, profile, privacy preferences, and account deletion request.
8. In-app notifications, push permission/preferences, command outcomes and authorized navigation from notification taps.
9. Report/block controls, an owned block list and a signed-out fictional demo with reset/exit.
10. Format catalogue/help, conditional setup and host-only group/team/role assignment, with reviewed/historical/house labels visible before joining.

## MVP acceptance criteria

- Deletion immediately removes active hosting/participation and precise-location access even if the cleanup worker is stopped; a retry cannot repeat the effects.
- Blocking a host/player prevents new shared interaction, applies the specified withdrawals and hides protected data. Unblocking does not restore a seat.
- An ongoing game's participant can leave after a host edit; the seat frees but joining remains closed once started.
- A delayed or rejected command, a lost push and a snapshot arriving after a newer event never display a false confirmation or remain labelled indefinitely as in progress.
- The demo is fully navigable with fictional roles/data and cannot reach production credentials or mutation endpoints.
- All format families have complete publish/discover/request/approve flows with the required materials/layout, including a custom declaration and an in-person device-play example. Role/format changes and safety withdrawals preserve seat/privacy invariants; no deck-legality or game-engine claim is implied.

- A player can discover a published session within a selected area, request a seat, and see its approval from a second account.
- A host can publish and cancel a session, approve requests, and inspect the current confirmed player count.
- Creating a four-player session with the default playing host leaves three available seats; choosing only to provide the table leaves four. The host can manage the session in both cases.
- Google and Apple work on iOS/Android via the accepted native/browser flows, enforce 18+ onboarding, resolve returning users and safely link/unlink providers. Matching names/emails never grant another profile access; logout/deletion revoke access immediately.
- A full, cancelled, completed, or already-started session cannot accept new participants.
- Concurrent approvals for the final seat result in exactly one successful approval.
- Location denial explains how to enable location and retry; nearby search never silently uses a default or manually selected destination.
- Discovery always exposes only the city. Unapproved players cannot retrieve exact location fields; approved participants can open the precise address and map in detail.
- Errors explain the next action, and core controls work with screen readers and larger text.
- Hosts can publish worldwide and players can discover sessions using simulated current-device positions in different countries and hemispheres.
- A host can change venue, format, time, and participation with confirmed players present; capacity stays valid and affected players receive an in-app notification with push delivery attempted.
- In-app notifications survive push denial, duplicates, and transient delivery errors; push content does not disclose the session's exact location.
- Critical flows and validation, provider-login, and session error states are available in English and Italian, with timezones preserved when changing language.
- Automated unit, integration, and E2E suites cover the applicable critical scenarios in the testing strategy and pass for the release candidate, including Android and iOS app journeys. Recorded real-provider and device smoke checks complement those suites.
