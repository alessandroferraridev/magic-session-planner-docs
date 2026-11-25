# Development roadmap

Deliver one usable flow at a time: create a session, find it nearby, then confirm a seat. Each step ends with a concrete completion check. The order below expresses dependencies; dates should be estimated after the initial scaffold and provider choices.

Unit, integration, and E2E tests are required for both API and app. Implement the relevant scenarios with each feature using the [testing strategy](./testing-strategy.md); testing is part of each step's completion criteria.

## Step 0 Resolve the product decisions

- Use the confirmed host approval policy and host participation choice at creation, defaulting to playing. Confirm who can host, initial formats, and rules for changing the host's playing choice later.
- Use the confirmed iOS and Android platforms through Expo, worldwide hosting and discovery, and English/Italian interface support. Confirm whether the pilot includes private homes; the pilot must not establish a regional product restriction.
- Use custom NestJS authentication for email magic link, Google, and Apple sign-in. Select authentication libraries and email delivery, and define account linking and session lifetimes. Select geocoding, API hosting, and managed PostgreSQL providers; verify PostGIS availability and budget.
- Apply the confirmed [Git workflow](./git-workflow.md), with main for releases, dev for staging, and pull requests for integration. Branch rulesets and dev defaults are configured in all three repositories; plan their CI setup.
- Resolve session cancellation, private address disclosure, retention periods, and an operator contact for reports.

**Complete when:** [Open decisions](./open-decisions.md) records the answers needed for implementation, and the MVP scope is agreed. Independent scaffolding can proceed using the documented defaults.

## Step 1 Establish the two projects

Repository setup:

- Work on feature branches derived from the existing protected `dev` branch in each repository.
- Use the configured pull request rules, merge strategies, and protected release tags. Add actual required CI check names only once those jobs exist, and version API and app releases independently.
- Configure validated `dev` changes for staging deployment or internal mobile builds, then tagged `main` releases for production delivery or store builds.
- Establish separate unit, integration, and E2E commands and CI jobs. Provision isolated test databases and Android/iOS test runners; add passing, stable job names to the GitHub rulesets once operational.

Backend tasks:

- Scaffold NestJS with strict TypeScript, linting, formatting, unit test support, and a supported Node.js version.
- Add environment validation, minimal health routes, global request validation, structured logs, and OpenAPI generation.
- Add Docker Compose with PostgreSQL and PostGIS, TypeORM configuration, and an initial extension migration.
- Commit an example environment file, local setup instructions, lockfile, and CI for lint, type checks, and tests.
- Configure Jest, NestJS TestingModule, PostgreSQL/PostGIS integration fixtures, and a Supertest HTTP E2E suite. Start with meaningful configuration or validation unit checks, migration integration checks, and API readiness over HTTP against the test database.

App tasks:

- Scaffold React Native with Expo, TypeScript, and routing.
- Establish navigation, accessible shared controls, theme tokens, loading states, and error states.
- Establish English and Italian translation catalogs, locale-aware formatting, a language setting, and a persisted preference. Keep locale independent of session geography and timezone.
- Configure API base URLs, TanStack Query, secure storage, environment examples, and basic CI.
- Document simulator and physical device setup, including API access over the local network.
- Configure Jest with jest-expo, React Native Testing Library, and a native E2E runner. Establish unit and integration examples and an installed-app smoke flow against the real test API; document Android and iOS execution prerequisites.

**Complete when:** both projects have runnable unit, integration, and E2E suites and pass the initial CI gates. The API reports database readiness, and Android and iOS test builds load a screen and call the real test API. Framework and test setup start here; they are not part of repository initialization.

## Step 2 Add identity and profiles

Depends on step 1, authentication library selection, email delivery setup, and Google and Apple application registration.

- Implement email magic link, Google, and Apple sign-in, app return routing, logout, token expiry, and secure token storage on iOS and Android.
- Localize authentication screens and email templates in English and Italian, including failures and expired-link recovery.
- Add local authentication identity, magic-link challenge, authentication session, and refresh token migrations. Implement single-use challenge consumption, application token issuance, refresh rotation and reuse detection, and device or global logout.
- Handle expired and reused magic links, resend requests, cancelled social sign-in, and failed callbacks. Define and implement account linking only after its policy is decided.
- Verify Google and Apple responses during sign-in, map them to local identities, and issue our application session. Validate our own access token signature, issuer, audience, and expiry for API requests.
- Implement profile read and update, account state checks, and authentication guards.
- Add authentication and authorization tests covering replay, expired challenges, refresh reuse, revoked sessions, cross-account access, and fixtures for host, player, and unrelated user accounts.

**Complete when:** all three sign-in methods work on iOS and Android development builds; two users can sign in independently, edit only their own profile, and recover cleanly from expired links and sessions. Logs contain no authentication tokens, magic-link tokens, authorization codes, or sensitive profile data.

## Step 3 Publish hosted sessions

Depends on step 2.

- Implement venue, protected venue location, session, and participation migrations and constraints.
- Add venue forms with public versus private visibility and a public preview.
- Support worldwide venue locations and international address formats, with timezone selection tied to the venue rather than the interface language or device timezone.
- Implement draft creation with hostParticipates defaulting to true, optional approved host membership, publishing, allowed edits, and cancellation.
- Show the host's playing choice in the creation form and seat preview; retain host management permissions when the host only provides the table.
- Build session detail and hosted session screens. Add realistic fixtures for public and private venues.
- Enforce time, capacity, ownership, and venue-edit restrictions on the API.
- Cover creation rules with unit tests, transactional persistence and API contracts with integration and HTTP E2E tests, and both host participation choices in app integration and native E2E flows.

**Complete when:** a host can publish and manage a session in either playing mode. A four-player table starts with three available seats when the host plays and four otherwise. The second account can see the session but cannot read the exact private address or edit it.

## Step 4 Discover nearby tables

Depends on step 3 and geocoder selection.

- Add geographic indexes and parameterized radius, date, format, and available-seat queries.
- Implement one-time foreground location lookup, permission explanation, and manual town selection.
- Allow manual search destinations worldwide and verify the selected geocoder across countries. Do not apply a launch-country filter; handle address lookup gaps with manual positioning.
- Build the nearby list, filters, pagination, refresh, approximate distance labels, and no-results state.
- Redact search coordinates from logging at the application and proxy layers.
- Measure geographic query plans with representative data and verify units and coordinate ordering.
- Add real PostGIS integration cases, HTTP E2E filters and privacy assertions, app integration state handling, and a native discovery flow including location denial and manual search.
- Include both interface locales, multiple hemispheres, high latitudes, and searches across the antimeridian in the relevant automated scenarios.

**Complete when:** test sessions inside and outside a selected radius appear correctly; full and cancelled sessions are excluded; manual search works with GPS disabled; private exact points never affect public radius filtering or ordering.

## Step 5 Request and confirm seats

Depends on step 4.

- Implement membership transitions and locking transactions for approvals and capacity changes.
- Build request, withdraw, approve, reject, and personal-session screens.
- Refresh server state after mutations and display useful conflict messages if the last seat is taken.
- Restrict exact location access to eligible confirmed players and the host.
- Add the scheduled completion job and cancellation history.
- Add unit transition cases, integration and HTTP E2E tests for final-seat races over independent connections, and native two-account request and approval flows with cancellation and conflict recovery.

**Complete when:** two accounts complete discovery through approval; concurrent approval of the last seat admits one player; leaving frees a seat; cancellation or withdrawal revokes private location access.

## Step 6 Prepare the pilot

Depends on steps 1 through 5.

- Run the complete regression matrix in [Quality and release](./quality-and-release.md), including the suites implemented in earlier steps, Android and iOS device journeys, real-provider sign-in smoke checks, and location denial.
- Add reporting, a minimal operator workflow, account deletion processing, and published privacy and support information.
- Deploy staging with HTTPS, managed PostGIS, migrations, backup configuration, log redaction, and error monitoring.
- Build distributable iOS and Android test versions and run an invited pilot with players or stores, covering English and Italian and representative locations and timezones without restricting global hosting.
- Record discovery failures, approval problems, accessibility issues, and query performance without collecting movement histories.

**Complete when:** critical defects are resolved, backups can be restored, a rollback is rehearsed, and pilot users can organize sessions without developer intervention.

## Step 7 Release and iterate

Depends on pilot results and launch readiness checks.

- Configure production environments, store listings, location permission descriptions, privacy declarations, and support contacts.
- Stabilize a release branch, validate its artifacts, merge through pull requests into `main`, tag the release, and return release fixes to `dev`. Use the documented hotfix flow for urgent production fixes.
- Require the applicable unit, integration, API HTTP E2E, and Android/iOS E2E gates to pass for the candidate revisions before release promotion.
- Run migration and API compatibility checks before deploying the production API and distributing the mobile app.
- Monitor errors, failed join attempts, empty searches, and latency. Investigate reports through the operator process.
- Prioritize improvements using pilot feedback: push notifications, more formats, maps, recurring sessions, waitlists, chat, store verification, and moderation tools.

**Complete when:** a player can install the app, use English or Italian, discover or host a real session at a chosen location worldwide, and complete the seat request and approval flow, with support and rollback procedures available.

## Dependency overview

```mermaid
flowchart LR
    Decisions[0 Product and providers] --> Foundations[1 Foundations]
    Foundations --> Identity[2 Identity]
    Identity --> Hosting[3 Session hosting]
    Hosting --> Discovery[4 Nearby discovery]
    Discovery --> Seats[5 Participation]
    Seats --> Pilot[6 Pilot]
    Pilot --> Release[7 Release]
```

The app can use agreed OpenAPI examples while API features are being implemented. Connect each feature to the staging API before calling it complete. If scope must shrink, postpone maps, push, and additional formats while preserving location privacy and safe seat allocation.
