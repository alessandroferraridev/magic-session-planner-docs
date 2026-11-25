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
| App E2E | Installed app on Android or iOS, connected to the real test API and database | Drive native UI and deep links; verify complete user journeys and server-side results without replacing our own API or database |

API HTTP E2E tests and mobile E2E tests are separate suites. Rendering a screen against mocked HTTP responses is an app integration test; it does not establish that the installed app works with the backend.

The API repository owns API unit, integration, and HTTP E2E suites. The app repository owns app unit, integration, and device E2E suites, including the launcher that selects a known API image or revision. Record both app and API revisions in full-system E2E results. The docs repository owns this shared specification and scenario traceability.

## Proposed implementation tools

| Scope | Proposed tools |
| --- | --- |
| API unit and integration | Jest and NestJS TestingModule; PostgreSQL/PostGIS containers for integration |
| API E2E | Jest, NestJS test application, and Supertest HTTP requests |
| App unit and integration | Jest with jest-expo and React Native Testing Library |
| App E2E | Maestro flows against native Expo development or test builds on Android and iOS |
| API smoke collection | Optional Postman CLI collection for deployed test environments; supplements the primary suites |

Confirm supported versions during scaffolding and pin dependencies. The required test levels are confirmed; exact library versions, device runner hosting, and CI budget remain setup choices. iOS simulator execution needs a macOS/Xcode runner; plan a local Mac or hosted runner alongside Android test infrastructure. Do not assume the Windows development machine can run an iOS simulator.

## Required scenario coverage

| Scenario | Unit coverage | Integration and API E2E coverage | App E2E acceptance |
| --- | --- | --- | --- |
| Authentication | Challenge expiry, session state, and identity decisions | Atomic magic-link consumption, replay rejection, social issuer/audience/signature checks, refresh rotation and reuse, device and global logout | Email link opens the app and establishes a real session; expired link and revoked session recover clearly |
| Session creation | Time ordering, capacity bounds, host participation default | Omitted or true hostParticipates creates one approved host membership; false creates none; transaction failure leaves no partial session | Publish a four-player table with three available seats by default and four when the host only provides the table |
| Host permissions | Ownership independent of playing status | Playing and non-playing hosts can manage their own session; another user cannot | A non-playing host publishes, reviews a request, and approves a player |
| Nearby discovery | Filter validation, units, and display rules | Real PostGIS queries for inside/outside/boundary cases, date/format filters, pagination, and availability | Set a location, find the expected table, and open its details; manual search works after location denial |
| Seat approval | Membership transitions and duplicate actions | A request remains pending until approved, retries are safe, approved count and availability stay consistent | Player requests, host approves, and player sees the confirmed seat using two accounts |
| Final seat | Capacity decision and conflict mapping | Concurrent approvals over separate database connections for one remaining seat produce one approval and one capacity conflict; total approved never exceeds capacity | The losing request shows a useful conflict and refreshes current availability |
| Location privacy | Location access decisions | Public responses omit exact private fields; discovery uses only the public point; authorized location access is revoked after withdrawal, cancellation, or completion | Private address appears only for an eligible approved player or host |
| Leaving and cancellation | Allowed transitions and lifecycle rules | Leaving frees a seat; cancellation blocks requests and approvals, including concurrent actions | Upcoming lists and details reflect withdrawal or cancellation after refresh |
| Time and lifecycle | UTC conversion boundaries, joining cutoff, and completion | Reject already-started sessions; verify timezone and daylight saving examples; completion is idempotent | Session details display the intended local time and timezone |
| Global geography | Full coordinate validation and independence from locale | Locations across countries and hemispheres, high latitudes, and antimeridian radius queries; no country allowlist or device-country restriction | Host and discover a table at a selected destination outside the device's area |
| English and Italian | Translation keys, locale selection and fallback, and formatting | Preferred locale persistence, both email templates, stable error codes, and unchanged UTC instants and venue timezones after locale changes | Run critical hosting, discovery, joining, and error recovery flows in en and it on Android and iOS; verify text expansion and language switching |
| Failures and accessibility | Input errors and status presentation | API validation and error contracts; app integration covers offline, timeout, stale cache, and expired session behavior | Essential actions remain understandable with errors, large text, and accessible controls; device permission failures have a recovery path |

Use synthetic host, player, unrelated user, public venue, and private venue fixtures. Add cases for valid, invalid, expired, forbidden, duplicate, and conflicting operations where the feature supports them. Product rules that remain open must be resolved before their acceptance tests are finalized.

## External services and native sign in

Unit and integration suites use controlled adapters for email, geocoding, Google, and Apple. API and full-system E2E suites run our own application and database for real; replace external services only at their boundaries. Use a local test mailbox for magic links and controlled issuer keys or provider responses for deterministic social authentication tests. Keep token verification active in those tests.

Automated social-auth tests using controlled provider responses do not verify the real Google or Apple consent UI. Before release, run and record smoke tests for all three actual sign-in methods on Android and iOS using dedicated test accounts. Native callbacks, permission prompts, and real-device behavior complement the automated suite.

Any test-only adapter must be selected by explicit test configuration, fail to activate in production, and never provide a public authentication bypass. Keep credentials outside the repository and redact tokens, email links, private addresses, and provider responses from reports and recordings.

## Test data and reproducibility

- Create an isolated test database or schema per run or parallel worker, with PostGIS enabled and production migrations applied. Do not replace it with SQLite or a mocked repository in database integration and API E2E suites.
- Verify migration installation from empty and upgrade from the previous supported schema. Use a clean fixture baseline for each test group and clean up on both success and failure.
- Validate the test environment and database destination before destructive fixture reset. Never reset production or a shared staging database.
- Freeze or inject time where appropriate, use known coordinates across hemispheres and the antimeridian, and include timezone and daylight saving examples. Exercise en and it without changing the same session's UTC instants or venue timezone. Avoid arbitrary sleeps and synchronize competing transactions so they actually overlap.
- Use independent connections for locking tests. Assert both operation results and final persisted state after all competing operations finish.
- Keep tests independent of execution order. Reset app secure storage, permissions, and query state between device scenarios; preserve backend data only for explicit multi-account journeys.

## Planned suite commands and layout

Introduce these scripts during scaffolding; each command must run its named suite independently and return a failing exit code when a test fails.

| Repository | Planned script names | Proposed test locations |
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

Publish sanitized test reports, coverage reports, and E2E failure diagnostics in CI. Report line and branch coverage; choose numeric thresholds after scaffolding establishes the initial baseline. Critical authentication, capacity, authorization, and location privacy scenarios above are required independently of the coverage percentage.

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
