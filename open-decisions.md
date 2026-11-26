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
| Authentication methods | Username and password replace email-link login; Google and Apple remain in the existing scope | Local login needs no email service or app-return link; social callback scope must be resolved separately |
| Authentication ownership | Custom authentication in the NestJS backend | Own application users, identity associations, sessions, and tokens; integrate directly with Google and Apple for social sign-in |
| Hosting providers | Vercel for NestJS and Neon for PostgreSQL/PostGIS | No email delivery provider is required; use bounded serverless jobs and isolated environments |
| Platforms | iOS and Android through Expo | Both platforms are required for development, testing, and distribution |
| Geographic availability | Worldwide session hosting and discovery | Accept valid locations globally; do not restrict use to a launch country or region |
| Interface languages | English and Italian | Localize the app in en and it; language preference is independent of session location |
| Git workflow | Gitflow using main, dev, feature, fix, release, and hotfix branches | main holds releases; dev integrates changes and feeds staging; use pull requests and independent repository versions |
| Testing | Unit tests, integration tests, and E2E tests for API and app | Implement suites from scaffolding and add feature coverage continuously; required failures block integration or release |
| Infrastructure budget | Zero infrastructure spending; budget only for app-store publication | Use free tiers and existing resources; no paid upgrades, overages, domain purchases, or paid CI/build services |
| Test infrastructure | GitHub Actions standard Linux/macOS runners, PostGIS containers, Jest/Supertest, jest-expo/RNTL, and Maestro | Public repositories, isolated data, zero paid runner/storage budget, and both native platforms |
| Coverage gates | 80% statements/functions/lines, 75% branches, and 90% branches for critical auth/participation/privacy modules | Required scenarios remain mandatory regardless of percentage; implement gates with the suites |

## Working assumptions and open decisions

The remaining defaults support planning only. They have not been confirmed.

| Decision | Proposed default | Effect on development |
| --- | --- | --- |
| Initial formats | Commander first | Keeps filters and forms small; format storage remains extensible |
| Authentication implementation details | Social scope, username model, recovery without email, account linking, session policy, and compatible libraries | Proposed defaults and the required product decisions are in the authentication document |
| Retention | Proposed category-specific periods, including 90-day session history and deletion within 30 days | See the operational proposal; durations require confirmation and implementation |
| Audience | Proposed initial 18+ personal-account audience worldwide | No store accounts; the age policy is a proposal rather than a confirmed restriction |
| Free delivery and backup feasibility | Validate included Workflow/Queue recovery and a private backup destination | Do not assume paid durable retention or 30-day managed database recovery is included |

## Questions still open

1. Should the first version focus on Commander or launch with multiple formats?
2. Which defaults in [Authentication decisions](./authentication-decisions.md) should be adopted for social login, recovery, usernames, linking, and session policy?
3. Should the proposed retention schedule and 18+ audience in the [operational document](./operational-proposal.md) be adopted?
4. Is there an existing private destination for any supplemental encrypted backups?

Zero infrastructure budget, test infrastructure, and coverage gates are confirmed. Retention, audience age policy, and the remaining service dependencies stay open. Keep scope, architecture, API, and roadmap aligned as answers arrive.
