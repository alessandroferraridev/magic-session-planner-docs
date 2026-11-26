# Quality and release plan

The critical guarantees are correct nearby results, reliable seat allocation, and controlled access to private locations. Verify these with the actual database and on physical mobile devices before inviting players.

Unit tests, integration tests, and E2E tests are mandatory for both API and app development. Follow the [testing strategy](./testing-strategy.md) for suite boundaries, implementation tools, required scenarios, CI gates, and feature completion criteria. Implement tests alongside each feature, starting during scaffolding.

## Verification evidence

[Testing strategy](./testing-strategy.md) owns the verification matrix, mandatory scenarios, tools, runners, coverage and CI gates. Release evidence identifies tested app/API revisions, passing required suites and recorded real-provider/device smoke checks. Optional Postman CLI smoke collections supplement those suites.

Before release verify screen-reader labels, focus after errors, large text, touch targets and status conveyed beyond color. Geographic tests use real PostgreSQL/PostGIS; cross-account authorization and city-only response assertions remain mandatory. External boundaries are controlled in automation; real Google/Apple consent and APNs/FCM delivery are checked on devices.

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
