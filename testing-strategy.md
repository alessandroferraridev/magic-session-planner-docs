# Unit Integration and E2E test strategy

Unit tests, integration tests, and end to end tests are required for both the NestJS API and the React Native app. Implement the relevant tests with each feature, starting with the project scaffold. A feature is complete when its behavior and failure paths are covered at the appropriate levels and the required suites pass.

The application repositories currently contain planning documentation rather than runnable projects. The tools, scripts, and CI gates below are implementation requirements; they are not existing test suites or configured GitHub checks.

## Test levels and responsibilities

| Repository and level | System under test | Dependencies and assertions |
| --- | --- | --- |
| API unit | One business rule, service, guard decision, or utility | Replace database and external adapters with controlled doubles; assert observable behavior and errors |
| API integration | Real feature modules, repositories, transactions, and migrations together | Use an isolated PostgreSQL database with PostGIS; verify persistence, constraints, locking, rollback, and geographic queries |
| API E2E | Complete NestJS application through HTTP | Use real routing, validation, authentication guards, services, and PostgreSQL/PostGIS; assert status, response contract, authorization, and persisted effects |
| App unit | Pure logic, form rules, and isolated component or hook behavior | Control native dependencies and time; assert user-visible state and accessible interactions |
| App integration | Screens, navigation, forms, query cache, and API client working together | Use controlled HTTP responses and native adapters; verify request construction, transitions, refresh, and recovery across components |
| App E2E | Installed app on Android or iOS, connected to the real test API and database | Drive native UI, notification taps, and the accepted native/browser social callbacks; verify complete user journeys and server-side results |

API HTTP E2E tests and mobile E2E tests are separate suites. Rendering a screen against mocked HTTP responses is an app integration test; it does not establish that the installed app works with the backend.

The API repository owns API unit, integration, and HTTP E2E suites. The app repository owns app unit, integration, and device E2E suites, including the launcher that selects a known API image or revision. Record both app and API revisions in full-system E2E results. The docs repository owns this shared specification and scenario traceability.

## Accepted implementation tools

| Scope | Accepted tools |
| --- | --- |
| API unit and integration | Jest and NestJS TestingModule; PostgreSQL/PostGIS containers for integration |
| API E2E | Jest, NestJS test application, and Supertest HTTP requests |
| App unit and integration | Jest with jest-expo and React Native Testing Library |
| App E2E | Maestro flows against native Expo development or test builds on Android and iOS |
| API smoke collection | Optional Postman CLI collection for deployed test environments; supplements the primary suites |

Confirm supported versions during scaffolding and pin dependencies. Use standard GitHub Actions Linux runners for API and Android work and standard macOS/Xcode runners for iOS, as accepted by the product owner. The public repositories enable the standard free compute model; paid runners, storage overages, and paid EAS builds are outside the zero infrastructure budget. The Windows development machine is not an iOS simulator runner.

Standard runner compute is free for the public repositories under [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions). Keep all three repositories public for this model, cancel superseded runs, use included artifact storage only and never expose production credentials to fork PRs. Small trusted Vercel/Neon staging smoke checks verify deployment compatibility; controlled issuer and APNs/FCM device release checks retain the boundaries described below. Real required status names are added to rulesets only after working jobs exist.

## Required scenario coverage

| Scenario | Unit coverage | Integration and API E2E coverage | App E2E acceptance |
| --- | --- | --- | --- |
| Authentication | Provider claim, session, identity, and public-name rules | Real signature/issuer/audience/nonce validation, replay rejection, identity uniqueness, throttling, 18+ onboarding, profile creation, returning-login resolution, 15-minute access/30-day inactivity/90-day absolute limits, immediate revocation, rotation/reuse, and logout | Google/Apple sign-in and onboarding work through the selected native/browser flows; cancellation, invalid credentials, session expiry, and provider recovery guidance are clear |
| Identity linking and unlinking | Ownership and last-linked-provider rules | Fresh verification links an identity to only its owner; matching emails or display names cannot merge accounts; unlink cannot remove the last usable identity and revokes all sessions; cross-account linking and stale proofs fail | Linking a second provider accesses the same profile, and losing all providers does not expose a local reset bypass |
| Session creation | Time ordering, capacity bounds, host participation default | Omitted or true hostParticipates creates one approved host membership; false creates none; transaction failure leaves no partial session | Publish a four-player table with three available seats by default and four when the host only provides the table |
| Host permissions | Ownership independent of playing status | Playing and non-playing hosts can manage their own session; another user cannot | A non-playing host publishes, reviews a request, and approves a player |
| Nearby discovery | Current-position state, filter validation, and city-only display | Real city-centroid PostGIS queries, same-city inclusion, neighboring-city boundaries, date/format filters, and availability | Simulate current device position, find a city-only card, and recover from permission denial using settings/retry |
| Seat approval | Membership transitions and duplicate actions | A request remains pending until approved, retries are safe, approved count and availability stay consistent | Player requests, host approves, and player sees the confirmed seat using two accounts |
| Final seat | Capacity decision and conflict mapping | Concurrent approvals and a host opting in compete under the same lock; one final seat is allocated once; invalid capacity edits fail atomically | A losing action shows a useful conflict with the response's current state |
| Location privacy | Host and approved-membership access decisions | Every discovery card and unapproved detail omit venue names and precise fields; use only canonical city data for filtering/ranking; revoke detail access on withdrawal/cancellation/completion | City-only list remains unchanged after approval; authorized detail loads the exact address and map |
| Session edits | Field validation, host toggles, and version rules | Change venue/time/format with approved players, reject stale edits, free or consume the host seat, keep other session snapshots unchanged, and record notifications atomically | Host edits a published session and opts out; the participant UI applies the newer event while host management remains available |
| Notifications | Event deduplication, absolute patch application, and version ordering | Durable ordinary commands, immediate safety transactions, terminal/expiry outcomes, idempotent replays, snapshot ordering, 202 recovery without polling, crash recovery and safe bounded dispatch | Initial inbox read followed by foreground push updates without another request; duplicate/out-of-order events do not corrupt state; background tap and re-entry recover missed events |
| Leaving and cancellation | Allowed transitions and lifecycle rules | Leaving frees a seat; cancellation blocks requests and approvals, including concurrent actions; withdrawal is allowed until end even after start | Upcoming lists and details reflect withdrawal or cancellation after refresh |
| Time and lifecycle | UTC conversion boundaries, joining cutoff, and completion | Reject already-started sessions; verify timezone and daylight saving examples; completion is idempotent and unrelated aggregate-version changes do not invalidate its completion generation | Session details display the intended local time and timezone |
| Global geography | Coordinate validation and independence from locale | Cities across hemispheres, high latitudes, and the antimeridian; no country allowlist; exact location never influences public results | Use simulated current-device locations worldwide while host venue creation remains globally available |
| English and Italian | Translation keys, locale selection/fallback, and formatting | Preferred locale persistence, localized signup/login/recovery messages, stable error codes, and unchanged UTC instants and venue timezones | Run critical registration, hosting, discovery, joining, and error recovery in en and it on Android/iOS; verify text expansion and language switching |
| Failures and accessibility | Input errors and status presentation | API validation and error contracts; app integration covers offline, timeout, stale cache, and expired session behavior | Essential actions remain understandable with errors, large text, and accessible controls; device permission failures have a recovery path |

API E2E also covers hidden drafts, DTO whitelisting, bounded pagination, request throttling, and an attempt to approve another host's applicant. Verify body-based discovery/address lookup never logs sensitive input. Use synthetic host, player, unrelated user, public venue, and private venue fixtures. Add cases for valid, invalid, expired, forbidden, duplicate, and conflicting operations where the feature supports them. Full format coverage is accepted; use the versioned catalogue and required configuration/host-declared fixtures in [Formats](./formats.md), without assuming Commander is the only launch format.

## Format coverage and configuration tests

All-format organizing support is required at first release. Test the entire catalogue as data plus every capability branch as behavior; an example five-format seed is insufficient. Card/deck legality and live digital game queues are external boundaries, not mocked guarantees.

| Scope | Required checks |
| --- | --- |
| Catalogue contract | Every one of the 26 official hub labels resolves to a documented canonical path; include all declared community/historical/custom entries, unique codes, valid revision references, en/it keys and declared source/review/lifecycle state |
| Aliases and authority | EDH/cEDH map to Commander with the right suggested intent/bracket; Brawl, Highlander and Commander 1v1 require disambiguation; Duel and MTGO lists remain separate; historic alias/date resolution does not invent current availability |
| Constructed profiles | Distinguish 60+ decks from human capacity, Standard/eternal pools, constructed Commander versus Limited Commander, Standard Brawl versus Arena Brawl, PDH, Oathbreaker and pointed/regional rules; null unverified guidance requires host declaration |
| Conditional validation | Required product manifests and quantity units, Cube owner/reference/return policy, custom pool/deck/layout notes, current/frozen rule references, commander/bracket compatibility and device/network/client information. Reject unknown fields, executable schemas, arbitrary URL fetches and oversized configuration |
| Teams and pods | One approved user occupies one seat, including a playing host. Four-person 2HG is distinct from a six-person Team Sealed match of three duels. Test several Commander/draft pods, odd duel rotation attendance, host-only mode, exclusive roles and foreign/overfilled/overlapping assignments |
| Readiness and safety | Publication validates planned slots while incomplete attendance is allowed. Assignment never reserves seats or bypasses host approval; concurrent assignment/approval/capacity edits use the same lock. Withdrawal/block/deletion/host opt-out remove roles even when a game becomes unready; no late join is enabled |
| Capacity boundaries | Explicit solo/custom/cooperative plan can have one human, ordinary duels cannot; valid two-person and sixteen-person layouts work, larger sessions fail under pilot limits. Deck size, boosters, schemes and bots never become seat counts |
| Query integration | Real PostgreSQL format/base/modifier/family joins combined with city-only PostGIS/date/seat filters; OR within facets and AND between facets. Custom text search and medium/intent filters stay parameterized and cannot leak exact locations |
| Revision changes | Publish from a pinned definition/profile, change the catalogue, then verify the session retains its snapshot. Explicit host upgrade is a material versioned edit, preserves members and clears only incompatible assignments. Older client schemas are read-only and unknown revisions are never silently coerced |
| Digital lifecycle | Physical device-play meetup has a protected physical venue; retired/unavailable client profile cannot masquerade as an active queue. Historical/house reconstruction is visibly declared; demo has zero real client, network or geocoding integration |
| Notification/cache | Compact format/own-role projections fit below 4 KiB in worst-case Unicode; no full custom notes/source text/location in push/history. A format/role change applies through existing revision ordering without per-push catalogue/API reads |
| Moderation and retention | Custom labels, rules and suggestions pass public-text controls; source suggestions remain private and cannot alter global IDs. Session/suggestion purge and restore retain correct non-personal revision links without resurrecting personal role data |

Unit tests cover schema selection, alias ambiguity, rule-profile compatibility, derived readiness and role validation. Database integration/API E2E exercise immutable revisions, facet queries, admission/locking, permissions and material-edit projections against real PostGIS. Native E2E runs EN/IT create-discover-request-approve/detail journeys for representative constructed Commander, team/limited, historical/custom, modifier/asymmetric and device-play recipes; parameterized contract tests cover the remaining seed entries. Include one-seat custom and host-only cases, source-unavailable labels, permission denial and concurrent safety changes. Existing 80/75/90 coverage gates remain; participation/privacy/assignment authorization is critical logic.

Parse the JSON fragments in [Formats](./formats.md#example-session-fragments) during documentation checks, and validate their schema/structural semantics once the catalogue implementation exists. Add a catalogue coverage job during scaffolding and require its real check name in rulesets once operational. No application test pass is claimed by this specification.

## Edge-case regression scenarios

The following cases close the specified product/contract gaps. They extend the required unit/integration/API E2E/native suites; they are not implemented or passing tests yet. Use the exact policies in the linked dedicated specifications as acceptance criteria.

| Area | Unit assertions | Integration and API E2E proof | Native E2E journey |
| --- | --- | --- | --- |
| Immediate account deletion | Account/host/player permission guards, cancellation and withdrawal effects | Stop queue/cleanup adapters; delete an account owning and joining active games; before acknowledgement verify revoked JWTs, cancelled hosting, freed seats, advanced versions, persisted inbox/tombstones and no location access. Race deletion with new requests/approvals/edits; failed transaction rolls back and repeated intent has no duplicate effect | Delete as a host/player, then use another account to verify hidden games and revoked map access; a late old-user response/push cannot repopulate caches |
| Complete command outcomes | State transitions, two-minute deadline and Outcome unknown UI without network timers | Delay execution beyond the ten-second response, then commit success, domain rejection or expiry; each creates one terminal actor outcome. Race execution/expiry under one lock; rejected/expired commands change no game. Persist success even if delivery fails; hidden/disabled actors cannot read results | Receive 202, apply a terminal push without a refetch; with push denied/lost show Check result and local Outcome unknown, then perform exactly one explicit read. Never auto-resubmit a new key or display a seat before success |
| Blocking and content moderation | Either-direction host/player/co-player eligibility, last-provider/auth separation, normalized text-rule outcomes | Block in both host/player directions and between two ordinary players; assert the specified withdrawals, other seats preserved, immediate private-data denial, generic notifications and no reverse block-list disclosure. Unblocking never restores seats. Reject prohibited public text with 422, test Unicode obfuscation and benign Magic card-name false positives, and preserve prior published revision; allow restricted report evidence | Block/report from a visible profile, confirm shared-game effects, manage own block list and appeal/edit rejected text; refused requests do not reveal who blocked whom |
| Changes after start | Leave-until-end rule, future-only joining and terminal history | Edit an ongoing game's material fields, preserve members, allow immediate player withdrawal and revoke location. A withdrawal before queued request execution marks that earlier command superseded and cannot be undone by its delayed worker. New requests/approvals remain denied unless the start is intentionally rescheduled into the future; end after now and capacity still hold | Host changes details after start; player sees the notice and leaves; freed seats do not reopen joining automatically |
| Snapshot/event concurrency | Per-key revision merge, tombstones, request/auth generations, buffer limits | Produce one repeatable-read snapshot watermark with concurrent recipient updates and paginated inbox. Verify read-state revisions and complete active-set semantics; a late create/approval cannot resurrect removed access | Buffer a newer push before an older snapshot returns, reverse event order across different keys, deliver a cancellation/read marker before old state, switch users mid-request and exceed 200 buffered envelopes. No hidden refetch; explicit refresh recovers flagged gaps |
| Admission and content bounds | Exact numeric/text/Unicode/error rules and independent safety buckets | Concurrent writes cannot exceed 5 active hosted games, 20 own memberships, 50 pending requests/game or 3 eligible interactive commands. Verify all configured rates/body limits and IP sharing. Exhaust ordinary/provider quotas and still execute bounded deletion/logout/block/withdrawal under their separate controls | Show actionable limit/content errors and Retry-After, preserve draft text, and keep safety actions usable when ordinary creation/search is limited |
| Idempotency retention | UUIDv7 age/future admission, keyed digest, logical replay expiry | Same key/action/input repeats one effect, changed input conflicts, old known keys replay within seven days. Advance past logical expiry and physical purge; original key is rejected and never creates another session. Validate concurrent duplicates, two-minute preparation expiry and earlier account purge | Lose the create response, retry its original key and receive the same game; expired keys require explicit new intent rather than automatic new-key retry |
| Review/demo access | Explicit demo namespace/roles, reset/exit and production-credential rejection | Release-configuration network spy proves demo sends zero external API/auth/push/geocoding/report requests and server refuses synthetic roles. Normal production auth remains real; apply the same domain validation to demo fixtures | In English/Italian on iOS/Android, use the public signed-out entry to complete both roles, map, notifications, pending/error, block/report and deletion screens; exit leaves no fixture state or real account |

Real social consent, push receipt delivery, actual pinned Workflow deployment, free-quota checks, backup restore and store acceptance of review access remain recorded release evidence. Mocked demo/issuer tests cannot certify them. Safety, blocking, capacity and deletion-authorization logic belongs to the existing 90% critical branch-coverage gate. Test notification read acknowledgement specifically: it updates the existing inbox resource and must not create a new unread operation notification recursively.

## External services and native sign in

Unit and integration suites control geocoding, push, and social issuer boundaries. API/full-system E2E run the real application, provider-token verification, and database; controlled issuer keys make credentials deterministic without disabling validation. The app has no local password or mailbox dependency.

Controlled issuer tests do not verify real provider consent UI. Before release, smoke-test Google and Apple on the agreed platforms with dedicated accounts. Test native callbacks, browser return where needed, permission prompts, and profile creation on devices. Do not assume an iOS-only SDK supplies Android support.

Any test adapter must use explicit configuration, fail to activate in production, and never provide an auth bypass. Keep credentials outside the repository and redact provider tokens/codes, private addresses, and provider responses from reports and recordings.

For notification automation, inject a synthetic foreground push event at the native adapter boundary and assert cache/inbox updates while an HTTP spy confirms no inbox or session refetch. Simulate duplicate, old, missing, and revoked-access events. A one-shot launch/re-entry snapshot is allowed for recovery; there is no timer-based client or notification-table polling. Device release checks separately exercise real APNs/FCM delivery and background notification taps.

## Retention and zero-cost operations coverage

Add integration/API E2E checks for 7-day exact snapshot/revision purge versus owner-saved venues, 30-day deletion, 90-day history/inbox cleanup, draft expiry, 24-month inactivity plus notice, two-minute command deadlines and separate auth deadlines, seven-day logical/eight-day physical command receipt expiry, owned block/cursor cleanup and deletion-ledger replay. Use injected time and bounded maintenance batches. Prove no precise data remains in delivery history and no expired job changes rescheduled sessions. Restore drills, quota/deployment checks, provider registrations and signed device builds are recorded acceptance tasks; they do not become invented passing CI checks.

## Test data and reproducibility

- Create an isolated test database or schema per run or parallel worker, with PostGIS enabled and production migrations applied. Do not replace it with SQLite or a mocked repository in database integration and API E2E suites.
- Verify migration installation from empty and upgrade from the previous supported schema. Use a clean fixture baseline for each test group and clean up on both success and failure.
- Validate the test environment and database destination before destructive fixture reset. Never reset production or a shared staging database.
- Freeze or inject time where appropriate, use known coordinates across hemispheres and the antimeridian, and include timezone and daylight saving examples. Exercise en and it without changing the same session's UTC instants or venue timezone. Avoid arbitrary sleeps and synchronize competing transactions so they actually overlap.
- Use independent connections for locking tests. Race approval against cancellation, withdrawal, completion and capacity edits, as well as other approvals/host opt-in. Assert both operation results and final persisted state after all competing operations finish.
- Keep tests independent of execution order. Reset app secure storage, permissions, and query state between device scenarios; preserve backend data only for explicit multi-account journeys.

## Planned suite commands and layout

Introduce these scripts during scaffolding; each command must run its named suite independently and return a failing exit code when a test fails.

| Repository | Planned script names | Accepted test locations |
| --- | --- | --- |
| API | test:unit, test:integration, test:e2e, test:coverage | Unit specs alongside source; test/integration and test/e2e for database and HTTP suites |
| App | test:unit, test:integration, test:e2e:android, test:e2e:ios, test:coverage | Unit and integration specs in separate test folders; Maestro flows in .maestro |

Document environment setup, database lifecycle, native build prerequisites, and example execution commands in each application repository. Run targeted suites locally while developing and the complete required suites in CI. Prefer assertions about behavior over implementation details or broad snapshots.

## CI and release gates

| Trigger | Required verification once test infrastructure exists |
| --- | --- |
| API PR into dev | Unit, database integration, and HTTP E2E suites; lint, type checks, and build |
| App PR into dev | Unit and integration suites, build validation, and Android device E2E smoke; add iOS smoke for native configuration, authentication, navigation, and permission changes |
| Integrated dev and release candidate | Full mobile E2E regression on Android and iOS against an identified API test artifact; API compatibility checks |
| PR into main or hotfix release | Applicable unit and integration suites, API HTTP E2E, full mobile E2E for mobile or shared-flow changes, and recorded real-provider/device release smoke checks |

Document-only changes use document checks instead of launching application suites. Keep relevant gate jobs present on every matching PR so conditional execution does not leave a required status pending. Promote a release only after the suites applicable to the changed components pass for the tested revisions; manual smoke checks supplement automated E2E rather than replacing them.

Publish sanitized reports and bounded failure diagnostics within included storage, retaining them for at most seven days and shorter when quota requires it. Accepted gates are 80% statements/functions/lines and 75% branches for owned logic, and 90% branches for authentication, capacity/participation, and location authorization once those modules exist. Generated clients and declarations may be excluded explicitly; business modules may not. [Operations](./operations.md) defines the zero-spending policy; runner and suite requirements are owned by this testing strategy. Critical scenarios remain mandatory independently of percentages.

Failed required tests block integration or release. Retain failure evidence when rerunning a flaky test; address the cause rather than treating repeated retries as a passing result. A missing required critical scenario remains unfinished work.

GitHub rulesets currently require pull requests but have no required CI checks because workflows do not exist yet. Add the real job names to the rulesets as the suites become operational; this specification does not enable those checks by itself.

## Feature completion criteria

1. Identify the affected unit, integration, and E2E scenarios before implementation.
2. Add meaningful tests at those levels with the feature; extend an existing E2E flow when that covers the new behavior clearly.
3. Add a regression test for a fixed behavioral defect at the level that can reproduce it reliably.
4. Pass the relevant local suites and required CI gates, including rejection and recovery cases.
5. Update scenario documentation and attach the validation results to the pull request. Include migration and cross-version checks when contracts or storage change.

Scaffolding establishes runnable suites and a real API/database and installed-app smoke flow. Feature steps add their tests continuously. The pilot runs the complete regression suite and real-device checks after the preceding steps have implemented their test coverage.

## Implementation references

- [NestJS testing](https://docs.nestjs.com/fundamentals/testing)
- [Expo testing with Jest](https://docs.expo.dev/develop/unit-testing/)
- [Expo device E2E testing with Maestro](https://docs.expo.dev/eas/workflows/examples/e2e-tests)
