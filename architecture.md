# Application architecture

Use a NestJS modular monolith backed by PostgreSQL with PostGIS, and a React Native application built with Expo and TypeScript. This keeps the initial system manageable while giving location search database support and allowing independent mobile and API releases.

## Proposed technology choices

| Area | Choice | Purpose |
| --- | --- | --- |
| Mobile | React Native, Expo, TypeScript, Expo Router | iOS and Android app with shared navigation and builds |
| Device location | Expo Location | Foreground permission handling and one-time position lookup |
| Server state | TanStack Query | Fetching, invalidation, pagination, and mutation feedback |
| Forms | React Hook Form and Zod | Client input validation and useful field errors |
| API | NestJS with TypeScript and REST | Feature modules, validation, guards, and service boundaries |
| API contract | NestJS OpenAPI generation | Documented endpoints and generated mobile client types |
| Database | PostgreSQL with PostGIS | Transactions, relational constraints, and geographic indexing |
| Persistence | TypeORM migrations and parameterized PostGIS queries | Explicit schema changes and location search |
| Authentication | Custom NestJS authentication with email magic links, Google, and Apple | Backend owns application accounts, sessions, access tokens, and refresh tokens |
| Local services | Docker Compose | Reproducible PostgreSQL and PostGIS development environment |
| CI and deployment | Provider selected in step 0 | Automated validation and separate staging and production environments |

Choose currently supported stable framework versions during scaffolding and commit each lockfile. Do not pin speculative versions in this plan. A map or geocoder provider must be selected before manual town search is implemented, considering coverage, cost, storage terms, and attribution.

## System boundaries

```mermaid
flowchart LR
    Player[React Native app] -->|Sign in and application bearer token over HTTPS| API[NestJS API and custom authentication]
    Player -->|Social sign in| Social[Google and Apple]
    API -->|Exchange and verify social identity| Social
    API -->|Deliver magic link| Email[Email delivery service]
    API --> DB[(PostgreSQL and PostGIS)]
    API --> Geo[Geocoding provider]
    API -. Optional later .-> Push[Push notification provider]
```

The API validates its own application access tokens. Google and Apple credentials are verified during social sign-in, then exchanged for our application session. Cache external public verification keys according to their providers' guidance. Store app tokens in device secure storage and enforce authorization rules on the API.

## Confirmed authentication methods

Support email magic links, Google sign-in, and Apple sign-in in the Expo app on iOS and Android. The NestJS authentication module owns local users, identity associations, login challenges, and application sessions. Google and Apple remain external identity providers for their respective sign-in methods; email delivery uses a separately selected service or SMTP setup.

- Generate cryptographically random magic-link tokens, store only their hashes, enforce expiry, and consume them atomically once. Opening the link must not expose a long-lived application credential in a URL.
- Integrate Google and Apple using maintained authentication libraries and their supported flows. Validate issuer, audience, signature, expiry, and transaction state or nonce; apply PKCE where the selected authorization-code flow supports it. Keep privileged credentials on the API.
- Issue short-lived application access tokens and rotating refresh tokens linked to a server-side authentication session. Store refresh token hashes, track rotation and reuse, and support revocation and logout from one or all devices.
- Rate-limit link requests, token consumption, and refresh operations. Use consistent link-request responses for existing and unknown addresses. Redact credentials from logs and analytics.
- Handle expired or reused links, cancelled social sign-in, failed callbacks, and revoked sessions in the app before updating signed-in state.

The email delivery service, library choices, token lifetimes, key rotation schedule, and account linking policy remain open. One local user can have multiple authentication identities; attaching an additional identity requires an explicitly defined ownership verification flow. Do not automatically merge accounts based only on matching email addresses while that policy is unresolved.

Use established libraries for cryptography and authentication protocols. Backend ownership includes maintaining the authentication implementation and its security tests. Reference the [NestJS authentication documentation](https://docs.nestjs.com/security/authentication), [OWASP email token guidance](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html), and [OWASP OAuth guidance](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html) during implementation.

## Backend modules

- `Auth`: magic-link challenges, Google and Apple integration, identity resolution, application token issuance and verification, refresh rotation, and authentication session revocation.
- `Users`: display profiles, preferences, and deletion requests.
- `Venues`: public search locations and protected private addresses.
- `Sessions`: drafts, the host's playing choice at creation, publishing, editing, cancellation, and lifecycle jobs.
- `Participation`: requests requiring host approval, optional approved host membership, withdrawal, and capacity transactions.
- `Discovery`: geographic filtering, distance sorting, and pagination.
- `Moderation`: basic reports and operator actions before public launch.
- `Health`: liveness, readiness, and dependency status.

Controllers validate requests, guards enforce access, and services own business rules. Database transactions span session and participation changes. Start without Redis, microservices, websockets, or a separate message broker. Add a transactional outbox if reliable asynchronous notifications become part of a later release.

## Mobile modules

Organize code by authentication, discovery, sessions, hosting, participation, and profile. Keep navigation routes small and place API access, secure token storage, and location services behind shared adapters.

The API is authoritative for capacity and membership. Invalidate session detail, discovery, and personal lists after participation changes. An optimistic UI must never show a confirmed seat before the server approves it. Avoid persistent caching of private venue details; clear sensitive query data when signing out or losing access.

## Geographic search

1. Accept latitude, longitude, radius in kilometers, date range, filters, and pagination. Validate coordinate ranges, cap radius at 100 km, page size at 50, and the search window at 90 days.
2. Store public discovery positions as PostGIS `geography(Point, 4326)` and create a GiST index. Construct points with longitude first and latitude second.
3. For public venues, use their published location. For private venues, use the host's selected neighborhood or area centroid as the discovery position; store the exact address and point separately.
4. Apply `ST_DWithin` to the discovery position using meters, then calculate and sort distance with `ST_Distance`. The radius filter and ordering must use the same public position. Never search against a secret point and merely round its returned distance.
5. Return rounded approximate distances, public location fields, and current available seats. Filter published future sessions using confirmed participation counts.
6. Begin with bounded offset pagination, defaulting to 20 results. Use stable secondary ordering by start time and session ID; results may move as capacity changes. Revisit cursor pagination after measuring demand.

Private venue search is approximate by design: neighborhood boundaries may affect whether a session appears near the edge of a radius. Show an approximate area label. Expose exact private details through a separate endpoint authorized for the host or an approved player, and revoke access after withdrawal, cancellation, or completion.

Device coordinates are request data for discovery. Do not maintain a history of player movement. Redact location query parameters from API, proxy, analytics, and error logs. Use foreground location on explicit actions; do not enable background tracking.

## Repository responsibilities

The API repository owns the schema, migrations, API, OpenAPI export, database fixtures, and API CI. The app repository owns the mobile experience, generated API client, device configuration, and mobile CI.

Publish a versioned OpenAPI artifact from backend CI, then update the app client in a reviewed app change. Keep additive changes compatible with older installed app versions. Use `/v1` for the initial API and maintain a staging API that development builds can reach from physical devices.

Use the [Git workflow](./git-workflow.md) in all three repositories. Validated `dev` changes feed staging; tagged releases on `main` identify production API artifacts and mobile release builds. API and app versions advance independently and must remain compatible with installed app versions.

Secrets belong in deployment secret stores or local ignored environment files. The app may contain the public API base URL and public OAuth client identifiers, but it must never contain database credentials, token-signing keys, OAuth client secrets, Apple signing credentials, or email service credentials.
