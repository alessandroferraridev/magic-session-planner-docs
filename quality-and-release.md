# Quality and release plan

The critical guarantees are correct nearby results, reliable seat allocation, and controlled access to private locations. Verify these with the actual database and on physical mobile devices before inviting players.

Unit tests, integration tests, and E2E tests are mandatory for both API and app development. Follow the [testing strategy](./testing-strategy.md) for suite boundaries, implementation tools, required scenarios, CI gates, and feature completion criteria. Implement tests alongside each feature, starting during scaffolding.

## Verification matrix

| Layer | Required checks |
| --- | --- |
| Backend unit tests | Session time validation, allowed lifecycle transitions, host participation default and both creation choices, membership rules, and host authorization independent of playing status |
| Database integration tests | Apply migrations on a fresh PostgreSQL and PostGIS instance; uniqueness, foreign keys, transaction rollback, locking, and idempotency |
| Geographic integration tests | Known locations inside, outside, and on the radius boundary; kilometers converted to meters; swapped coordinates; private areas; invalid coordinates |
| Concurrency tests | Multiple approvals compete for one seat; approval races with cancellation, withdrawal, completion, and capacity edits |
| API E2E tests | Complete NestJS app over HTTP with PostgreSQL/PostGIS: token validation, hidden drafts, field validation, hostParticipates and seat counts, pagination, rate limiting, ownership, and protected location access |
| Custom authentication tests | Magic-link expiry and atomic single consumption, concurrent token redemption, consistent link-request responses, social issuer and audience checks, state or nonce mismatch, refresh rotation and reuse, and device or global session revocation |
| Mobile unit tests | Form rules, isolated component and hook behavior, validation errors, and accessible status presentation |
| Mobile integration tests | Screens, navigation, forms, query cache, and API client together: discovery states, membership status, failed actions, and refresh after a conflict |
| Device end to end tests | Email magic link, Google, and Apple sign-in on iOS and Android; expired or reused links, cancelled social sign-in, failed callbacks; two-account host and join flow in both host modes, denied permission, GPS off, manual location, poor connectivity, and token expiry |
| Accessibility checks | Screen reader labels, focus after errors, large text, touch targets, and status communicated beyond color |

Run geographic tests against PostgreSQL with PostGIS, not an SQLite substitute. Include a malicious request that tries to approve another host's applicant and discovery response assertions that reject any exact private location fields.

The proposed tools are Jest and NestJS TestingModule for backend unit and integration tests, Supertest for API HTTP E2E, Jest with jest-expo and React Native Testing Library for app unit and integration tests, and Maestro for native E2E. Confirm compatible versions and device runners during scaffolding. Postman CLI is available for an optional deployed API smoke collection and supplements the primary suites. Use isolated test environments and injected test credentials, with no secrets committed.

Automated E2E uses the real app, API, and database with controlled external-service boundaries. Real Google and Apple sign-in and device permission behavior also need recorded release smoke checks, as described in the testing strategy.

## Privacy and abuse controls

- Request foreground location when needed and provide manual search. Do not request background tracking.
- Store private venue addresses separately, restrict reads server-side, and prevent sensitive response caching and analytics capture.
- Validate payloads and enforce ownership, bounded search ranges, per-user and per-IP rate limits, and request size limits.
- Store magic-link and refresh token hashes, restrict credential access, and redact authentication payloads and callback parameters from logs. Verify identity ownership before linking accounts once its policy is defined.
- Limit public participant information to display names and approved membership. Pending requests remain visible only to their owner and host.
- Define retention periods before the pilot. Deletion processing must cancel future hosted sessions, withdraw future memberships, revoke access, and delete or anonymize profile and venue information according to the agreed retention policy.
- Restrict operator access, record moderation actions, and define a report response process. Confirm launch audience and age requirements before public distribution.

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
- The pilot confirms all three sign-in methods on iOS and Android, both host participation choices, public and private venue flows, location denial, cancellation, the final-seat race, and an app update against the previous API contract.
- Public venue hosts understand that publishing does not reserve a store table; operators can act on reported or unsafe sessions.

## Deferred capabilities

Push notifications need device token lifecycle management, retries, preferences, and delivery monitoring. Add an outbox and background dispatcher when that work begins. Chat, ratings, payments, and public store verification each need their own product and abuse requirements before development.
