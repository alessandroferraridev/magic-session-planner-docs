# Open product decisions

The requested technical foundations are fixed: NestJS for the backend, React Native for the app, separate Git repositories, and geographic discovery of available Magic: The Gathering tables.

## Confirmed decisions

| Decision | Confirmed choice | Effect on development |
| --- | --- | --- |
| Joining policy | Host approval | Players request a seat; only approval confirms participation and occupies a seat |
| Host participation | Chosen when creating the session; participating is the default | A playing host occupies one seat; a host who only provides the table occupies none and still manages the session |
| Who can host | Anyone using a personal profile | All profiles represent individuals; organization accounts and store hosting features are outside scope |
| Session editing | The host can edit all session business fields, including their own participation | Allow venue, time, format, capacity, and participation changes after publication; preserve ownership, valid state, and seat capacity |
| Discovery origin | Current foreground device position | No manually selected search destination; permission and location failures have explicit recovery states |
| Location disclosure and map | Discovery list shows only the city; approved participants see the precise location and map in session detail | Protect exact addresses and coordinates on the API; hosts can inspect their own session location |
| Notifications | In-app and push in the MVP, without polling | Load an initial snapshot; push events update local UI state directly. Persist authoritative events and use event-triggered delivery, with one-shot recovery on app entry |
| Authentication methods | Email magic link, Google, and Apple | All three sign-in methods are in scope |
| Authentication ownership | Custom authentication in the NestJS backend | Own application users, identity associations, sessions, and tokens; integrate directly with Google and Apple for social sign-in |
| Hosting providers | Vercel for NestJS, Neon for PostgreSQL/PostGIS, Resend for email | Use bounded serverless jobs and database connections; separate production and staging resources |
| Platforms | iOS and Android through Expo | Both platforms are required for development, testing, and distribution |
| Geographic availability | Worldwide session hosting and discovery | Accept valid locations globally; do not restrict use to a launch country or region |
| Interface languages | English and Italian | Localize the app and user-facing authentication emails in en and it; language preference is independent of session location |
| Git workflow | Gitflow using main, dev, feature, fix, release, and hotfix branches | main holds releases; dev integrates changes and feeds staging; use pull requests and independent repository versions |
| Testing | Unit tests, integration tests, and E2E tests for API and app | Implement suites from scaffolding and add feature coverage continuously; required failures block integration or release |

## Working assumptions and open decisions

The remaining defaults support planning only. They have not been confirmed.

| Decision | Proposed default | Effect on development |
| --- | --- | --- |
| Initial formats | Commander first | Keeps filters and forms small; format storage remains extensible |
| Authentication implementation details | Libraries, token lifetimes, and account linking rules to be selected | Resend is confirmed for delivery; local identity linking rules remain open |
| Infrastructure budget | Proposed target 60 to 80 USD/month, review threshold 100 USD | Event-driven delivery avoids recurring notification scans; see the operational proposal and usage assumptions |
| Retention | Proposed category-specific periods, including 90-day session history and deletion within 30 days | See the operational proposal; durations require confirmation and implementation |
| Audience | Proposed initial 18+ personal-account audience worldwide | No store accounts; the age policy is a proposal rather than a confirmed restriction |
| Test infrastructure details | Proposed GitHub Actions standard runners, local PostGIS containers, Maestro, and coverage gates | See the operational proposal for runner allocation, thresholds, costs, and confirmation status |

## Questions still open

1. Should the first version focus on Commander or launch with multiple formats?
2. Which authentication libraries should be used, and how should accounts be linked across sign-in methods?
3. Which proposed budget, retention, audience, and test infrastructure settings should be adopted from the [operational proposal](./operational-proposal.md)?

The confirmed product decisions and provider choices above can guide implementation. Budget, retention, audience age policy, and detailed test infrastructure remain proposals until accepted. Keep scope, architecture, API, and roadmap aligned as answers arrive.
