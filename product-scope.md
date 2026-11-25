# Product scope

The first release helps players arrange an in-person Magic: The Gathering game near a chosen location. A table represents one scheduled session at a venue with a fixed player capacity. It does not represent a store's live inventory of physical furniture or guarantee that a store has reserved a table.

## Confirmed MVP decisions

- A host approves seat requests. Pending requests do not reserve seats.
- At creation, the host chooses whether to play or only provide the table. Playing is selected by default. A playing host occupies one seat; a host who only provides the table occupies none. A four-player session therefore begins with three or four available seats respectively.
- Authentication supports email magic links, Google, and Apple. The provider remains undecided.
- The app targets iOS and Android through Expo.

## Working MVP assumptions

These are proposed defaults until the questions in [Open decisions](./open-decisions.md) are answered.

- Players and store representatives can host sessions at public venues or private locations. Store verification is a later capability; the MVP must not imply a store endorsement.
- Commander is the first supported format. Store the format as an extensible value for later additions.
- The app uses foreground location only.
- Account access is required to browse and participate. Payments, chat, competitive matchmaking, deck management, ratings, and tournament brackets are outside the MVP.

## Main user flows

### Find a table

1. Sign in and open nearby sessions.
2. Explain why location is useful, then request foreground permission when the player chooses to use their location.
3. Fetch the current position once. Allow a manually selected town or map point if permission is denied, location services are off, or the player wants another area.
4. Show a list of future, published sessions with seats available. Default to a 10 km search radius, adjustable from 1 to 100 km, and a date window of the next seven days.
5. Show format, start time, approximate distance, area or public venue name, available seats, and host name. Filter by date, radius, format, and player count; sort by distance, then start time and ID.
6. Open session details and request one seat. Clearly show pending, approved, rejected, and cancelled outcomes.

The list must handle loading, no results, stale results, location failure, and network failure. Show when a manual area is being used and allow changing it. A map view is a later enhancement.

### Host a session

1. Choose a venue or create one, including its public area and location visibility.
2. Enter a title, format, start and end time, timezone, total player capacity, language, accessibility notes, and a short description of the expected play style or deck strength. Choose whether to join as a player or only provide the table; the player option is selected by default and the available-seat preview reflects it.
3. Validate the draft and publish it. For private venues, show the host the exact address separately from the public area preview.
4. Review requests and approve players while capacity remains. The host sees confirmed players and pending requests.
5. Edit allowed details or cancel the session. Once a non-host player is confirmed, changing venue, format, or start time requires cancelling and creating a replacement in the MVP; descriptions and end time can still be updated with clear change information.

Capacity may be increased. It cannot fall below the current confirmed player count. Publishing a session does not reserve space with a third-party venue; the host is responsible for any venue booking.

### Manage participation

Players can see hosted sessions and their pending or confirmed sessions, withdraw a pending request, and leave a confirmed session before it starts. Leaving immediately frees a seat. Hosts see and manage their hosted sessions whether or not they play. The rules for changing a host's playing choice after creation remain open; ownership transfer and removing confirmed players are deferred working assumptions.

The app shows authoritative status when reopened or refreshed. Push alerts for approvals and cancellations can follow the core MVP; participation must work without push delivery.

## Session rules

| Rule | MVP behavior |
| --- | --- |
| Lifecycle | Draft, published, cancelled, completed |
| Full table | Derived from confirmed participation and capacity; not a separate lifecycle state |
| Availability | Published, starts in the future, and confirmed players below capacity |
| Join request | One request per player per session; every non-host player needs host approval |
| Host participation | Chosen at creation, default true; create an approved host membership only when the host plays |
| Host permissions | Session ownership grants management access independently of player membership |
| Duplicate action | Retried requests must not create duplicate membership or allocate another seat |
| Completion | At the scheduled end time, complete an active session through an idempotent scheduled job |
| Joining cutoff | Requests and approvals stop at the session start time |
| Location visibility | Exact private address is visible only to the host, whether playing or not, and active approved players in an active session |
| Time | Store UTC instants and the venue's IANA timezone; display the timezone on session details |

## Required mobile screens

1. Sign in through email magic link, Google, or Apple, including authentication return and error states.
2. Location explanation, permission handling, and manual area selection.
3. Nearby session list and filters.
4. Session detail and participation state.
5. Create and edit session, including venue visibility preview and the creation option for host participation.
6. Hosted sessions and request management.
7. My upcoming sessions, profile, privacy preferences, and account deletion request.

## MVP acceptance criteria

- A player can discover a published session within a selected area, request a seat, and see its approval from a second account.
- A host can publish and cancel a session, approve requests, and inspect the current confirmed player count.
- Creating a four-player session with the default playing host leaves three available seats; choosing only to provide the table leaves four. The host can manage the session in both cases.
- Players can sign in through each of the three methods on iOS and Android development builds; the authentication flow returns to the app and handles failure clearly.
- A full, cancelled, completed, or already-started session cannot accept new participants.
- Concurrent approvals for the final seat result in exactly one successful approval.
- Location denial never blocks manually searching a town or area.
- Nearby lists and non-member session details never expose a private venue's exact address or coordinates.
- Errors explain the next action, and core controls work with screen readers and larger text.
