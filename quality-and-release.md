# Quality and release plan

The critical guarantees are correct nearby results, reliable seat allocation, and controlled access to private locations. Verify these with the actual database and on physical mobile devices before inviting players.

Unit tests, integration tests, and E2E tests are mandatory for both API and app development. Follow the [testing strategy](./testing-strategy.md) for suite boundaries, implementation tools, required scenarios, CI gates, and feature completion criteria. Implement tests alongside each feature, starting during scaffolding.

## Verification matrix

| Layer | Required checks |
| --- | --- |
| Backend unit tests | Session time validation, allowed lifecycle transitions, host participation default and both creation choices, membership rules, and host authorization independent of playing status |
| Database integration tests | Apply migrations on a fresh PostgreSQL and PostGIS instance; uniqueness, foreign keys, transaction rollback, locking, and idempotency |
| Geographic integration tests | Body-based search without logged coordinates; same-city inclusion and neighboring-city radius cases across hemispheres; city centroids, antimeridian and high-latitude cases; exact venue coordinates never affect pre-approval results |
| Concurrency tests | Multiple approvals compete for one seat; approval races with cancellation, withdrawal, completion, and capacity edits |
| API E2E tests | Complete NestJS app over HTTP with PostgreSQL/PostGIS: token validation, hidden drafts, field validation, hostParticipates and seat counts, pagination, rate limiting, ownership, and protected location access |
| Custom authentication tests | Google/Apple claim/signature/nonce validation, replay and identity collisions, shared throttling, safe profile creation/linking, display-name separation, and refresh/session revocation |
| Mobile unit tests | Form rules, isolated component and hook behavior, validation errors, and accessible status presentation |
| Mobile integration tests | Screens, navigation, forms, query cache, and API client together: discovery states, membership status, failed actions, and refresh after a conflict |
| Session edit tests | Venue/time/format changes with approved players, version conflicts, host opt-out/opt-in, capacity races, unaffected neighboring sessions, and transactional recipient notifications |
| Notification tests | Initial snapshot, direct push-cache patches, no polling/per-push reads, duplicates/order, crash recovery, denied permission, token invalidation, and authorized navigation from taps |
| Localization tests | English/Italian completeness, persisted locale, signup/login/recovery messages, locale-aware formatting, and session timezone independence |
| Device end to end tests | Google/Apple onboarding and returning sign-in on agreed platforms; cancellation and callbacks; two-account approval/map; host editing; current location recovery; push/cache and re-entry |
| Accessibility checks | Screen reader labels, focus after errors, large text, touch targets, and status communicated beyond color |

Run geographic tests against PostgreSQL with PostGIS, not an SQLite substitute. Include a malicious request that tries to approve another host's applicant and discovery response assertions that reject any exact private location fields.

The accepted tools are Jest and NestJS TestingModule for backend unit/integration, Supertest for API HTTP E2E, Jest with jest-expo and React Native Testing Library for app unit/integration, and Maestro for native E2E. Use standard GitHub Actions Linux/macOS runners for the public repositories and included storage only. Confirm compatible versions during scaffolding and implement the accepted coverage gates from the testing strategy. Postman CLI can supplement the primary suites with an optional deployed smoke collection. Use isolated fixtures and credentials with no secrets committed.

Automated E2E uses the real app, API, and database with controlled external-service boundaries. Real Google and Apple sign-in and device permission behavior also need recorded release smoke checks, as described in the testing strategy.

## Privacy and abuse controls

- Request foreground location for nearby discovery, and offer settings and retry when unavailable. Host venue entry is separate; no manual discovery destination or background tracking.
- Store precise session locations separately for all venue types. Discovery and unapproved detail expose only the city; exact map data requires host or approved-participant authorization.
- Validate payloads and enforce ownership, bounded search ranges, per-user and per-IP rate limits, and request size limits.
- Store refresh token hashes, restrict provider credential access, and redact auth payloads. Verify identity ownership before linking; public names and matching emails cannot grant access. No local credential or reset-code store exists.
- Limit public participant information to display names and approved membership. Pending requests remain visible only to their owner and host.
- Implement the accepted retention schedule before the pilot. Deletion cancels active hosted sessions, withdraws active memberships, immediately revokes access and purges/anonymizes personal records within 30 days, including backup expiry and deletion-ledger replay on restore.
- Implement the restricted operator CLI/audit and daily pilot report review with a 72-hour first-response target. Require the accepted 18+ declaration and versioned policy acknowledgement; private homes use the same city-only discovery rules.

## Performance and operational targets

Treat these as initial pilot targets to measure, not existing guarantees. With 10,000 future sessions, 100,000 participation rows, 20 results per page, and 20 concurrent discovery users, target API discovery latency below 500 ms at the 95th percentile in staging. Target participation mutations below one second at the 95th percentile under representative pilot traffic.

Inspect query plans to confirm index use and avoid loading all sessions into memory. Monitor slow queries, API errors, connection saturation, failed auth, and capacity conflicts. Do not log coordinates, private addresses, provider credentials, tokens/codes, or unnecessary profile data.

## Release checklist

- Follow the [Git workflow](./git-workflow.md): pull requests for integration, protected `main` and `dev`, a tested release candidate, a version tag, and release or hotfix corrections returned to `dev`.
- CI passes linting, type checks, required unit, integration, and E2E suites, API contract generation, and build validation for the changed application components.
- Staging/production use separate Frankfurt resources and credentials, HTTPS and validated environments. Exercise Neon instant restore and weekly/pre-migration encrypted exports to the private operator PC directory, with deletion-ledger replay, maximum 30-day retention and no public CI copies.
- Migrations are tested from both a fresh database and the previous release. Production schema changes remain compatible with the currently installed app and previous API release.
- Secrets and privileged provider keys are absent from source control and app bundles.
- Plans/runners respect zero spending with no paid upgrade, converting trial, or overage fallback. Verify free quotas and social callback setup with available hostnames; no email-sending domain is required.
- Auth passes provider verification/replay checks, throttling, identity/profile mapping, accepted linking, refresh rotation, and revocation. Validate native and browser callbacks for Google and Apple on the agreed platforms.
- Exercise database restore and app/API rollback, including invalidating restored credentials and applying deletion/retention before reopening traffic. Require a successful encrypted export before production migrations; destructive rollback follows the recovery runbook.
- Publish bilingual privacy/community/support and deletion-request information on the free docs GitHub Pages site, with a private support contact. Account deletion in-app, store permission text, third-party map processing and privacy declarations match actual behavior.
- The pilot confirms Google/Apple login, host changes, city-only discovery, approved maps, no-polling push updates, location recovery, capacity races, and API compatibility.
- Verify English/Italian registration, credential and recovery messages, accessibility, and longer text. Device-position fixtures cover countries, hemispheres, and timezones while discovery stays tied to current position.
- Individual hosts are responsible for third-party venue bookings; operators can act on reported or unsafe sessions.

## Deferred capabilities

Chat, ratings, and payments need separate product and abuse requirements before development. In-app and push notifications, precise approved-detail maps, and editable host participation are part of the MVP. Store or organization accounts are outside the personal-profile product scope.
