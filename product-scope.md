# Product scope

The first release helps individuals arrange an in-person Magic: The Gathering game anywhere in the world, with nearby discovery around the device's current location. A table represents one scheduled session with a fixed player capacity, hosted by a person.

## Confirmed MVP decisions

- A host approves seat requests. Pending requests do not reserve seats.
- Anyone with a personal profile can host. Profiles represent individuals; store or organization accounts and business hosting features are outside scope.
- At creation, the host chooses whether to play or only provide the table. Playing is selected by default. A playing host occupies one seat; a host who only provides the table occupies none. A four-player session therefore begins with three or four available seats respectively.
- The NestJS backend manages authentication through email magic links, Google, and Apple sign-in. It owns application accounts and sessions; email delivery and account linking details remain open.
- The app targets iOS and Android through Expo.
- Session hosting and discovery are available globally, without a launch-country or region restriction.
- Nearby discovery uses the current device position. The discovery list and unapproved session detail expose only the city; an approved participant sees the exact location and map in session detail. Hosts can inspect their own exact locations.
- In-app and push notifications are required in the MVP. Fetch once on app entry and update the UI from push payloads without timed polling or a new read for every notification.
- Hosts can edit all session business fields and change their own playing choice after creation. Capacity and ownership invariants still apply.
- The interface supports English and Italian. Language selection is independent of where a session is hosted.
- API and app development must include unit, integration, and E2E tests, implemented alongside their features from the initial scaffold. See the [testing strategy](./testing-strategy.md).

## Working MVP assumptions

These are proposed defaults until the questions in [Open decisions](./open-decisions.md) are answered.

- Commander is the first supported format. Store the format as an extensible value for later additions.
- The app uses foreground location only.
- Account access is required to browse and participate. Payments, chat, competitive matchmaking, deck management, ratings, and tournament brackets are outside the MVP.

## Global availability and interface languages

Hosts can create a session at any valid location worldwide. Players discover sessions around their current device position wherever they are. This does not introduce a manually selected search destination or a country restriction. Venue positioning during hosting is a separate action from choosing the discovery origin.

Provide English and Italian translations for navigation, forms, validation, empty and error states, and user-facing authentication emails. Session titles, descriptions, and venue names remain user content without automatic translation. A session's play language is separate metadata from the interface language.

Use the venue's IANA timezone to interpret and display session time, retaining UTC instants for storage. Choosing English or Italian does not change a session's location or timezone. An invited pilot is a validation exercise and does not define a geographic restriction for the product.

## Main user flows

### Find a table

1. Sign in and open nearby sessions.
2. Explain why location is useful, then request foreground permission when the player chooses to use their location.
3. Fetch the current foreground device position. If permission is denied, location services are off, or lookup fails, show settings and retry actions. Nearby discovery waits for a usable current position; personal upcoming and hosted session lists remain usable.
4. Show a list of future, published sessions with seats available. Default to a 10 km search radius, adjustable from 1 to 100 km, and a date window of the next seven days.
5. Show format, start time, city, available seats, and host name. The location label is the city only, without venue name, address, map, or exact coordinates. Filter by date, radius, format, and player count; rank geographic results using city-level discovery data.
6. Open session details and request one seat. Clearly show pending, approved, rejected, and cancelled outcomes.

The list must handle loading, no results, stale results, location failure, and network failure. Once approval is confirmed, session detail loads the protected exact address and map. The list remains city-only even for approved players. The precise map is part of the MVP detail flow.

### Host a session

1. Choose a venue or create one, including its public area and location visibility.
2. Enter a title, format, start and end time, timezone, total player capacity, language, accessibility notes, and a short description of the expected play style or deck strength. Choose whether to join as a player or only provide the table; the player option is selected by default and the available-seat preview reflects it.
3. Validate the draft and publish it. For private venues, show the host the exact address separately from the public area preview.
4. Review requests and approve players while capacity remains. The host sees confirmed players and pending requests.
5. Edit title, description, venue, location, format, time, capacity, and playing choice, or cancel the session. Existing confirmed players do not force cancellation and recreation. Persist a new session version and notify affected pending and approved players about material changes, with no exact address in notification payloads.

Capacity can be changed but cannot fall below the resulting confirmed player count. A playing host opting out frees one seat; opting in requires one available seat, or a valid capacity increase in the same edit. These changes are transactional. Publishing does not reserve a third-party venue; the individual host remains responsible for any venue booking.

### Manage participation

Players can see hosted sessions and their pending or confirmed sessions, withdraw a pending request, and leave a confirmed session before it starts. Leaving immediately frees a seat. Hosts keep management rights when they opt out of playing and can opt back in while a seat is available. Ownership transfer and removing another confirmed player are separate from editing session fields and remain deferred working assumptions.

The app loads an initial notification and personal-session snapshot. Push listeners update local UI state directly using safe versioned event payloads for requests, decisions, cancellations, and material edits, without a backend read for every push. Opening protected precise detail remains an authorized user action. Reconcile once when entering or returning to the app, or on explicit refresh, to recover missed pushes. Denied push permission does not block participation or erase persisted notifications; immediate updates then wait for a one-shot refresh.

## Session rules

| Rule | MVP behavior |
| --- | --- |
| Lifecycle | Draft, published, cancelled, completed |
| Full table | Derived from confirmed participation and capacity; not a separate lifecycle state |
| Availability | Published, starts in the future, and confirmed players below capacity |
| Join request | One request per player per session; every non-host player needs host approval |
| Host participation | Default true at creation; host can later opt out to free a seat or opt in when capacity allows |
| Host permissions | Session ownership grants management access independently of player membership |
| Duplicate action | Retried requests must not create duplicate membership or allocate another seat |
| Completion | At the scheduled end time, complete an active session through an idempotent scheduled job |
| Joining cutoff | Requests and approvals stop at the session start time |
| Location visibility | City only in discovery and before approval; exact address and map only for the host and active approved participants in active session detail |
| Time | Store UTC instants and the venue's IANA timezone; display the timezone on session details |

## Required mobile screens

1. Sign in through email magic link, Google, or Apple, including authentication return and error states.
2. Current location explanation, permission handling, settings recovery, and retry.
3. Nearby session list and filters.
4. Session detail and participation state, including protected precise location and map after approval.
5. Create and edit all session business fields, including a city-only preview and editable host participation.
6. Hosted sessions and request management.
7. My upcoming sessions, profile, privacy preferences, and account deletion request.
8. In-app notifications, push permission and preferences, and notification deep links.

## MVP acceptance criteria

- A player can discover a published session within a selected area, request a seat, and see its approval from a second account.
- A host can publish and cancel a session, approve requests, and inspect the current confirmed player count.
- Creating a four-player session with the default playing host leaves three available seats; choosing only to provide the table leaves four. The host can manage the session in both cases.
- Players can sign in through each of the three methods on iOS and Android development builds; the authentication flow returns to the app and handles failure clearly.
- A full, cancelled, completed, or already-started session cannot accept new participants.
- Concurrent approvals for the final seat result in exactly one successful approval.
- Location denial explains how to enable location and retry; nearby search never silently uses a default or manually selected destination.
- Discovery always exposes only the city. Unapproved players cannot retrieve exact location fields; approved participants can open the precise address and map in detail.
- Errors explain the next action, and core controls work with screen readers and larger text.
- Hosts can publish worldwide and players can discover sessions using simulated current-device positions in different countries and hemispheres.
- A host can change venue, format, time, and participation with confirmed players present; capacity stays valid and affected players receive an in-app notification with push delivery attempted.
- In-app notifications survive push denial, duplicates, and transient delivery errors; push content does not disclose the session's exact location.
- Critical flows, validation and error states, and authentication emails are available in both English and Italian, with session timezones preserved when changing language.
- Automated unit, integration, and E2E suites cover the applicable critical scenarios in the testing strategy and pass for the release candidate, including Android and iOS app journeys. Recorded real-provider and device smoke checks complement those suites.
