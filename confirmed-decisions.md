# Confirmed product and implementation decisions

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
| Authentication methods | Google and Apple only | No local username/password, recovery codes, or sending service; native SDK/callback setup still applies to social login |
| Public identity | Public display name is separate from sign-in identity | No login username in the MVP; internal UUID and provider issuer/subject identify the user |
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

## Decisions finalized on 26 November 2026

| Decision | Accepted choice | Implementation reference |
| --- | --- | --- |
| Audience and homes | 18+ by self-declaration, personal accounts worldwide; private homes included | [Operating policy](./operational-proposal.md) |
| Account linking | Explicit fresh verification of both providers; no email/name merging, no last-provider unlink, no account merge | [Authentication](./authentication-decisions.md) |
| Auth sessions | 15-minute access, rotating refresh with 30-day inactivity/90-day absolute expiry; immediate server-side revocation checks | [Authentication](./authentication-decisions.md) |
| Social platform implementation | Native Google on both platforms, native Apple on iOS and secure browser Apple on Android; fixed identifiers and callback contract | [Authentication](./authentication-decisions.md) |
| Maps/geocoding | Geoapify Free and MapLibre React Native, attribution and quota limits | [Operating policy](./operational-proposal.md) |
| Durable notifications | Vercel Workflow/Nest integration and Expo push, durable command before mutation, bounded retries and no polling | [Delivery and recovery](./delivery-and-recovery.md) |
| Deployment | Separate staging/production in Frankfurt using provider HTTPS hostnames; personal/non-commercial free tiers | [Operating policy](./operational-proposal.md) |
| Retention | 90-day session/inbox history, exact session snapshots 7 days, account purge within 30 days, inactive accounts after 24 months plus notice | [Full category schedule](./operational-proposal.md#retention-schedule) |
| Backup feasibility | Included limited Neon instant restore plus weekly/pre-migration encrypted local manual exports; maximum 30-day copy retention, no paid automated backup promise | [Backup decision](./operational-proposal.md#backup-and-recovery-decision) |
| Remaining implementation defaults | Confirmed stack, authenticated browsing, foreground-only location, capacity 2-16, terminal history, private operator CLI and pilot guardrails | [Audit](./decision-audit.md) |

The only unresolved choice is recorded in [Open decisions](./open-decisions.md).
