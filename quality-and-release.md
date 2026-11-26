# Quality and release plan

The critical guarantees are correct nearby results, reliable seat allocation, and controlled access to private locations. Verify these with the actual database and on physical mobile devices before inviting players.

Unit tests, integration tests, and E2E tests are mandatory for both API and app development. Follow the [testing strategy](./testing-strategy.md) for suite boundaries, implementation tools, required scenarios, CI gates, and feature completion criteria. Implement tests alongside each feature, starting during scaffolding.

## Verification matrix

| Layer | Required checks |
| --- | --- |
| Backend unit tests | Session time validation, allowed lifecycle transitions, host participation default and both creation choices, membership rules, and host authorization independent of playing status |
| Database integration tests | Apply migrations on a fresh PostgreSQL and PostGIS instance; uniqueness, foreign keys, transaction rollback, locking, and idempotency |
| Geographic integration tests | Same-city inclusion and neighboring-city radius cases across hemispheres; city centroids, antimeridian and high-latitude cases; exact venue coordinates never affect pre-approval results |
| Concurrency tests | Multiple approvals compete for one seat; approval races with cancellation, withdrawal, completion, and capacity edits |
| API E2E tests | Complete NestJS app over HTTP with PostgreSQL/PostGIS: token validation, hidden drafts, field validation, hostParticipates and seat counts, pagination, rate limiting, ownership, and protected location access |
| Custom authentication tests | Magic-link expiry and atomic single consumption, concurrent token redemption, consistent link-request responses, social issuer and audience checks, state or nonce mismatch, refresh rotation and reuse, and device or global session revocation |
| Mobile unit tests | Form rules, isolated component and hook behavior, validation errors, and accessible status presentation |
| Mobile integration tests | Screens, navigation, forms, query cache, and API client together: discovery states, membership status, failed actions, and refresh after a conflict |
| Session edit tests | Venue/time/format changes with approved players, version conflicts, host opt-out/opt-in, capacity races, unaffected neighboring sessions, and transactional recipient notifications |
| Notification tests | Initial snapshot, direct push-driven cache patches, no polling/per-push reads, duplicate and out-of-order handling, crash recovery, denied permission, token invalidation, and safe deep links |
| Localization tests | English and Italian translation completeness, persisted locale, validation and error messages, email templates, locale-aware formatting, and session timezone independence |
| Device end to end tests | Three sign-in methods on iOS/Android; expired links and failed callbacks; two-account approval and exact map; editable host participation; location denial and retry; push permission, foreground UI update, background tap, and missed-event reconciliation |
| Accessibility checks | Screen reader labels, focus after errors, large text, touch targets, and status communicated beyond color |

Run geographic tests against PostgreSQL with PostGIS, not an SQLite substitute. Include a malicious request that tries to approve another host's applicant and discovery response assertions that reject any exact private location fields.

The proposed tools are Jest and NestJS TestingModule for backend unit and integration tests, Supertest for API HTTP E2E, Jest with jest-expo and React Native Testing Library for app unit and integration tests, and Maestro for native E2E. Confirm compatible versions and device runners during scaffolding. Postman CLI is available for an optional deployed API smoke collection and supplements the primary suites. Use isolated test environments and injected test credentials, with no secrets committed.

Automated E2E uses the real app, API, and database with controlled external-service boundaries. Real Google and Apple sign-in and device permission behavior also need recorded release smoke checks, as described in the testing strategy.

## Privacy and abuse controls

- Request foreground location for nearby discovery, and offer settings and retry when unavailable. Host venue entry is separate; no manual discovery destination or background tracking.
- Store precise session locations separately for all venue types. Discovery and unapproved detail expose only the city; exact map data requires host or approved-participant authorization.
- Validate payloads and enforce ownership, bounded search ranges, per-user and per-IP rate limits, and request size limits.
- Store magic-link and refresh token hashes, restrict credential access, and redact authentication payloads and callback parameters from logs. Verify identity ownership before linking accounts once its policy is defined.
- Limit public participant information to display names and approved membership. Pending requests remain visible only to their owner and host.
- Confirm the proposed retention schedule before the pilot. Deletion processing cancels future hosted sessions, withdraws memberships, revokes access, and purges personal data according to the accepted policy, including backup expiry and restore handling.
- Restrict operator access, record moderation actions, and define a report response process. The 18+ audience and age declaration are proposals in the operational document, not yet confirmed restrictions.

## Performance and operational targets

Treat these as initial pilot targets to measure, not existing guarantees. With 10,000 future sessions, 100,000 participation rows, 20 results per page, and 20 concurrent discovery users, target API discovery latency below 500 ms at the 95th percentile in staging. Target participation mutations below one second at the 95th percentile under representative pilot traffic.

Inspect query plans to confirm geographic index use and avoid loading all sessions into application memory. Monitor slow queries, API error rates, connection pool saturation, failed authentication, and capacity conflicts. Do not log player coordinates, private addresses, bearer tokens, magic-link tokens, authorization codes, or email addresses.

## Release checklist

- Follow the [Git workflow](./git-workflow.md): pull requests for integration, protected `main` and `dev`, a tested release candidate, a version tag, and release or hotfix corrections returned to `dev`.
- CI passes linting, type checks, required unit, integration, and E2E suites, API contract generation, and build validation for the changed application components.
- Staging and production use separate credentials and databases, HTTPS, validated environments, and managed backups.
- Migrations are tested from both a fresh database and the previous release. Production schema changes remain compatible with the currently installed app and previous API release.
- Secrets and privileged provider keys are absent from source control and app bundles.
- Custom authentication has passing challenge replay, social identity validation, refresh token rotation, and revocation checks. Email delivery and mobile callbacks work in the target environment.
- Database restore and application rollback procedures are exercised; destructive schema rollback uses a documented recovery procedure.
- Privacy policy, support contact, account deletion route, store location permission text, and privacy declarations match actual behavior.
- The pilot confirms both platforms, three sign-in methods, host participation changes, city-only discovery, approved exact maps, no-polling push-driven updates, current-location denial/recovery, capacity races, and compatibility with the previous API contract.
- Verify English and Italian, email templates, accessibility, and longer text. Use device position fixtures across countries, hemispheres, and timezones; discovery remains tied to current position.
- Individual hosts are responsible for third-party venue bookings; operators can act on reported or unsafe sessions.

## Deferred capabilities

Chat, ratings, and payments need separate product and abuse requirements before development. In-app and push notifications, precise approved-detail maps, and editable host participation are part of the MVP. Store or organization accounts are outside the personal-profile product scope.
