# Operations

Status: accepted on 26 November 2026. The MVP is a personal, non-commercial project with zero infrastructure spending beyond app-store publication. These policies are decisions; resources, credentials, workflows, and restore drills still require implementation.

## Deployment and environments

Deploy NestJS on Vercel and connect to Neon over TLS with a bounded connection pool. Use a direct migration connection where the selected migration tooling needs one, and execute migrations as a controlled release step rather than on every request. Validate the production PostgreSQL/PostGIS versions and transaction behavior in staging. [Vercel NestJS support](https://vercel.com/docs/frameworks/backend/nestjs), [Neon pooling](https://neon.com/docs/connect/connection-pooling), and [Neon PostGIS](https://neon.com/docs/extensions/postgis) guide the setup.

Use separate Vercel projects for staging and production, with separate Neon projects or equivalently isolated resources and credentials. Staging can use its own production deployment of `dev` to exercise scheduled jobs; preview deployments must not be assumed to run the production schedule. Gate production delivery from `main` on the release process and keep test notifications confined to test accounts. Verify current [scheduler behavior](https://vercel.com/docs/schedules/cron-jobs/manage-cron-jobs) during implementation.

Durable command, notification and scheduled-job execution is specified in [Delivery and recovery](./delivery-and-recovery.md). A failed compatibility or quota acceptance check blocks release within the zero budget. Provider registrations, policy publication and staging evidence are implementation tasks in the [roadmap](./development-roadmap.md).

## Free service baseline

| Service | Accepted choice and boundary |
| --- | --- |
| API | Vercel Hobby, personal/non-commercial use only; separate staging/production projects and supplied HTTPS hostnames |
| Database | Neon Free PostgreSQL/PostGIS, separate staging/production projects; pooled application connection, direct controlled migrations and scale-to-zero |
| Event execution | Vercel Workflow SDK with `@workflow/nest`, using included Hobby Workflows/Queues/Functions; no separate broker or notification polling |
| Push | Expo push service with APNs/FCM credentials; best-effort delivery and an authoritative PostgreSQL inbox |
| Geocoding/maps | Geoapify Free for worldwide address/city resolution and tiles; MapLibre React Native with Geoapify `osm-bright` style and visible Geoapify/data-source attribution |
| Authentication | Direct Google/Apple integrations and NestJS-owned sessions as specified in [Authentication](./authentication.md) |
| Native builds/CI | Local Android builds and standard GitHub Actions Linux/macOS runners for public repos; signing on trusted release jobs only; no paid EAS or larger runners |
| Backup/monitoring | Included Neon instant restore plus manual encrypted local exports; provider dashboards and redacted logs, no paid observability service |
| Public information | Bilingual privacy/community/support and deletion-request information on the free docs GitHub Pages site; actual authenticated account deletion in-app |

No subscriptions, usage-billed upgrades, converting trials, new domains, paid storage, or automatic overages are authorized. Remain on non-billing free plans. Initial compute region is Frankfurt: Vercel `fra1` and Neon AWS `eu-central-1`. This is not a geographic product restriction or a worldwide latency promise.

Limits checked on 26 November 2026: [Vercel Hobby](https://vercel.com/docs/plans/hobby.md), [Vercel terms](https://vercel.com/legal/terms), [Workflow pricing](https://vercel.com/docs/workflows/pricing), [Queues pricing](https://vercel.com/docs/queues/pricing), [Neon plans](https://neon.com/docs/introduction/plans), [Expo push](https://docs.expo.dev/push-notifications/faq/), and [Geoapify pricing/attribution](https://www.geoapify.com/pricing/). Recheck actual account allowances before deployment; these are limits, not app capacity promises.

Plan around Neon Free's 100 CU-hours/project/month, 1 GB PostgreSQL/project and 5 GB public transfer/project/month. Workflow Hobby includes 50,000 events/month and 1 GB written, without extended retained workflow data; Queues includes 1,000,000 operations. Function/queue use also counts. Geoapify Free includes 3,000 credits/day and five requests/second; tile and address use share credits. Bound retries/fan-out, debounce host address lookup and avoid speculative map loading. Maps need an Expo development build rather than Expo Go: [MapLibre setup](https://maplibre.org/maplibre-react-native/docs/setup/expo/), [Geoapify integration](https://apidocs.geoapify.com/how-to/maps/react-native-maplibre-map).

Review dashboards weekly/before releases and daily during the pilot. Use included alerts where available; Neon Free does not include paid-plan spending alerts. At 70% usage investigate and reduce optional/staging activity; at 85% pause pilot invitations and expensive exports/test traffic; at exhaustion return explicit unavailable/retry states. Never upgrade silently or discard accepted user data. Admission controls reject new commands before acceptance when execution resources are unavailable. Start with an invitation-only load-limited pilot, without a promised user count or uptime SLA. Monetization is outside scope and would require reviewing Hobby eligibility.

App map keys are public and limited to map usage/quota; private geocoding and privileged keys stay server-side. Explain that map providers receive viewport requests when authorized users open maps. No exact addresses in shared caches. Persist only needed canonical city/centroid and host-owned venue data under provider storage/attribution terms. Allow manual venue pin/address input, but require canonical city and IANA timezone before publishing; failed city resolution blocks publication until retry without adding manual discovery destinations.

Store publication is the funded exception: [Apple Developer Program](https://developer.apple.com/programs/enroll/) and [Google Play registration](https://support.google.com/googleplay/android-developer/answer/6112435?hl=en). No extra hardware or paid build plan is assumed.

## Moderation and support operations

Audience, age declaration, permitted host profiles and private-home support are defined in [Product scope](./product-scope.md#audience-and-hosting).

Provide the published private support route and operator CLI for report review, account disable and unsafe-session cancellation. Restrict operator credentials, record actor/reason/time and never derive operator permission from public profile ownership. Review reports daily during the pilot, targeting first response within 72 hours without promising 24/7 coverage. Sensitive reports must not use public GitHub issues. Publish the actual private support contact and policy pages before invitations.

## Retention schedule

These are adopted product limits, not statutory durations or a worldwide compliance claim. Publish purposes/vendor behavior before production collection. [Storage-limitation guidance](https://commission.europa.eu/law/law-topic/data-protection/information-business-and-organisations/principles-gdpr_en) informs minimization. Earlier account deletion overrides normal retention except a documented narrow security/legal hold.

| Category | Accepted period | Required behavior |
| --- | --- | --- |
| Discovery device position | Request lifetime | No movement history; redact coordinates/query strings from logs and analytics |
| Account/linked identities | While used | Deletion immediately disables access, cancels active hosted games and withdraws active memberships; purge/anonymize personal records within 30 days |
| Inactive/incomplete account | 24 months since last authenticated use | In-app/push notice where available, 30 days to return, then deletion; no guaranteed email notice or background keep-alive |
| Saved venue/precise saved location | While explicitly saved by an active owner | Owner-only; delete on owner request/account deletion. Independent from session snapshots |
| Unpublished session draft | 30 days after last host edit | Delete draft/private snapshot |
| Session/participation history | 90 days after end/cancellation | Delete personal history; no indefinite movement analytics |
| Exact session address/map and prior revisions | 7 days after end/cancellation | Deny access at completion/cancellation; purge all precise revisions separately from city-level history |
| In-app notification | 90 days from creation | Paginated inbox, no precise address; earlier deletion applies |
| Successful delivery/command result | 7 days after completion | Minimal IDs/outcomes and safe projection; erase private command input on commit/terminal rejection |
| Failed delivery diagnostics | 30 days from terminal failure | Redacted codes/IDs, no user content or provider secrets; bounded retries |
| Prepared/unaccepted command | 24 hours | No business effects; expire encrypted input and reject late execution |
| Pending command input | Until execution, maximum 24 hours | Encrypted/owner-restricted; expired command cannot mutate data |
| Push association | Until logout/deletion or 90 days without foreground use | Remove on logout, disable invalid tokens; foreground registration renews last-seen |
| Auth sessions/refresh hashes | 30-day inactivity, 90-day absolute lifetime | Minimal expired/revoked session and consumed-token tombstones kept 30 further days for reuse detection; no raw refresh secret |
| Auth transactions/completion codes | 10 minutes / 60 seconds | Single-use; reject by timestamp even before daily cleanup |
| Throttle buckets | Maximum 24 hours | Salted IP/installation buckets, no raw request/token history |
| App/security logs | Maximum 30 days | Shorter provider defaults allowed; no precise location, credentials, bodies or unnecessary profile data |
| Closed reports/operator audit | 180 days after closure/action | Restricted access, open-case review every 30 days; justify exceptions |
| Deletion ledger | From deletion request until 35 days after final purge | Minimal deleted UUID/request/purge times, encrypted local recovery copy; prevent resurrection from backups |
| Backups/exports | Maximum 30 days from creation | Private/encrypted/bounded; replay deletion ledger and retention before serving a restore |
| Test fixtures/failure artifacts | Fixtures per run; artifacts maximum 7 days | Synthetic only, within included storage; no production copies in public CI |

Maintenance rechecks the current lifecycle/version and end/cancel time before purging edited sessions; obsolete timers cannot erase a rescheduled active session. Saved venues have their own purpose. Remove orphaned exact-location revisions with drafts/history. Daily batches select records whose expiry falls before the next scheduled run, purging slightly early rather than exceeding maximum periods; account deletion cannot exceed 30 days. Application authorization also checks expiry immediately, independently of physical cleanup.

Provider-managed backups/logs may not be individually erasable. Document those limits; row deletion does not instantly erase backups. On restore keep traffic disabled, replay the current deletion ledger and time-based retention, invalidate restored auth/push credentials, and only then reopen. Never revive a deleted account or send old queued notifications.

## Backup and recovery

Use included Neon instant restore first: up to six hours bounded by change volume, a restore window rather than a six-hour RPO guarantee. Automated snapshot schedules are paid; a manual-snapshot count limit does not establish free snapshot storage. The zero-cost baseline does not depend on managed snapshots or a 30-day managed recovery service. See [Neon recovery/snapshot limits](https://neon.com/docs/introduction/plans).

Supplement with a private non-synced directory on the maintainer's existing PC outside all Git repositories. Manually run a custom-format `pg_dump` weekly and immediately before production migrations, streamed into `age` public-key encryption without a plaintext dump. Keep the decryption key separately. Retain at most four weekly exports plus one pre-migration copy, expire each within 30 days and cap local storage at 5 GB. Check disk space and Neon transfer quota first; failed mandatory pre-migration export blocks release. This is a manual operational duty, not unattended backup automation.

Update an encrypted copy of the minimal deletion ledger after each completed deletion and before exports. Consult the live ledger when available; if neither a current ledger nor recovery-safe deletion record is available, keep the restored service offline. Never put production dumps/deletion records in public CI artifacts, repository history, or shared drives. No off-site storage account/capacity is assumed.

Before the pilot and every three months, restore into a private isolated database, apply ledger/retention, validate schema/relationships, and record only a redacted drill result. Pilot recovery target: one working day once operator/copy are available; fallback export data may be seven days old if weekly exports succeeded. Missed duties, hardware loss, provider failure, or exhausted quota can increase loss/outage. These limitations are accepted for this zero-budget pilot; independent automated disaster recovery is outside the funded MVP, not an open product decision.

## CI costs

The [testing strategy](./testing-strategy.md#accepted-implementation-tools) owns runner choices, public-repository constraints, test artifacts, coverage and CI gates. Apply those constraints within the zero-spending policy above.
