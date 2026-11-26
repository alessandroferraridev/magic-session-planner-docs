# Operating constraints retention audience and test infrastructure

The product owner has selected Vercel for NestJS and Neon for PostgreSQL/PostGIS. Local signup/login uses username/password without an email delivery service or sending domain. Infrastructure spending is zero beyond store publication. Test infrastructure and coverage gates are accepted; retention, audience, recovery/social policies, and the durable delivery adapter still have open choices. No service account or billing setting is provisioned by this documentation.

## Confirmed zero infrastructure budget

Use free tiers and existing resources only, with no subscription, paid overage, trial that converts to payment, new domain purchase, larger runner, or paid build service. App-store publication is the only funded expense. Worldwide location support remains a product requirement; unlimited load or paid availability guarantees are not implied. Begin with a controlled pilot sized to measured free-tier usage rather than a promised user count.

| Item | Incremental cost | Free baseline and limits |
| --- | --- | --- |
| Vercel | 0 | Hobby, subject to personal/non-commercial eligibility and free compute/transfer limits |
| Durable delivery | 0 | Only included Hobby Workflow/Queue allowances; validate the SDK and no-charge quota behavior before adoption |
| Neon | 0 | Free plan: 100 CU-hours/project/month, 1 GB PostgreSQL storage/project, and 5 GB public transfer/project/month |
| Push transport | 0 | Expo push service candidate; sends still consume the API/database's included resources |
| Mapping and geocoding | 0 | A provider with a usable free tier and attribution; reject paid overages rather than assuming a paid map allowance |
| API hostname | 0 | Use the supplied vercel.app HTTPS hostname; local credentials need no sending domain |
| Backup and monitoring | 0 | Included provider features and existing private storage; no paid log retention or backup destination |
| CI and native builds | 0 | Standard GitHub Actions runners for the public repositories, within included storage; no paid EAS or larger runner |
| Total infrastructure spending | 0 | Quota exhaustion can reduce availability; no automatic paid fallback |

Free-tier checks dated 26 November 2026: [Vercel Hobby](https://vercel.com/docs/plans/hobby.md), [Vercel terms](https://vercel.com/legal/terms), [Neon Free limits](https://neon.com/docs/introduction/plans), [Expo push fees](https://docs.expo.dev/push-notifications/faq/), and [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions).

Vercel Hobby is for personal or non-commercial use. Keep that eligibility explicit when choosing the initial launch model. Neon Free suspends compute or restricts operations when its resource limits are exhausted; storage remains a continuous limit. Use small pools, scale-to-zero, isolated low-usage staging resources, and no notification polling or always-on database scans. Monitor quotas and provide clear unavailable/retry states; do not silently upgrade or discard accepted user data to stay under a quota. Neon supports [PostGIS](https://neon.com/docs/extensions/postgis).

The [Hobby Workflow allowance](https://vercel.com/docs/workflows/pricing) includes 50,000 events/month and 1 GB written data; extended retained Workflow data is not available on Hobby. [Queues](https://vercel.com/docs/queues/pricing) includes an allowance of 1,000,000 operations, with message sizing/options affecting consumption. Function execution also uses its own quota. These allowances are feasibility inputs, not permission to enable paid metering. Validate durable recovery, retention, and no-charge limit behavior together before selecting the adapter.

Apply alerts before quota exhaustion and configure zero paid usage wherever the provider exposes that control. Do not attach an automatic upgrade or paid marketplace integration. If a required feature cannot fit a free plan or existing resource, record the blocked capability and propose a zero-cost design adjustment; it is not authorized to incur a small charge instead.

For geocoding, [Geoapify's free tier](https://www.geoapify.com/pricing/) is a candidate with 3,000 credits/day and required attribution. Select a detail-map option that also fits free use without a paid billing fallback. Limit requests and handle quota failure explicitly; hosting location input remains separate from a manual discovery destination.

## Authentication without email delivery

Username/password signup and login remove the sending-service and verified-domain dependency from local authentication. No email is required for the proposed local account model. Password hashing, throttling, credential changes, recovery, and session revocation are now backend responsibilities within the free runtime's capacity.

Finalize recovery without email and the retained social provider/callback scope in [Authentication decisions](./authentication-decisions.md). Do not assume that removing email-link login removes Google/Apple callbacks or provides a password recovery mechanism. Recovery codes are a proposal; they require secure generation, hashing, single-use consumption, and a clear lost-code policy if adopted.

The funded exception is store publication: [Apple Developer Program](https://developer.apple.com/programs/enroll/) lists 99 USD/year, with regional pricing, and [Google Play registration](https://support.google.com/googleplay/android-developer/answer/6112435?hl=en) lists 25 USD once. Existing memberships can cover these costs. Use CI for native builds; no paid EAS subscription or new test hardware is included.

## Proposed deployment arrangement

- Use separate Vercel projects for staging and production, and separately credentialed Neon resources. Place the API compute near its database; an initial European location is an infrastructure proposal, not a limit on where users can host sessions.
- Keep application authentication custom in NestJS; selecting Neon does not select Neon's managed authentication product.
- Use event-triggered durable notification work rather than polling the notification table. Prototype the recovery boundary between database commit and delivery with the selected Vercel Workflow or Queue SDK before finalizing it.
- Load an initial notification snapshot and patch foreground UI state from safe push payloads. Reconcile once on app entry/foreground return or explicit refresh to recover missed events; do not refetch for every push.
- Configure session-end work for the relevant session version and revalidate after edits. Retention cleanup is a bounded daily maintenance task, not a notification refresh loop.
- Keep staging pushes confined to test recipients. Preview deployments use synthetic data and never inherit production credentials.
- Size both environments within free quotas, reuse included provider hostnames, and keep the staged delivery adapter's resource use bounded. No production-to-public-CI database copies are allowed.

## Proposed audience

Launch for adult Magic players aged 18 and over worldwide, using personal profiles only. This simplifies the first release's in-person and private-location flows. The age restriction is proposed and has not been adopted as a confirmed product decision.

Use an explicit age declaration and policy acknowledgement rather than collecting a full birth date or identity document for ordinary registration. This is not a claim of verified age. Support casual players and hosts, in English and Italian, without business accounts, store administration, payments, or professional tournament tooling. Keep reporting and a documented support response process in the MVP. Select the initial format separately; Commander remains a proposal.

## Proposed retention schedule

Durations below are product and operational proposals, not statutory retention periods or a claim of worldwide legal compliance. Confirm the purpose of each category and publish the accepted policy before collecting production data. The [European Commission's storage-limitation guidance](https://commission.europa.eu/law/law-topic/data-protection/information-business-and-organisations/principles-gdpr_en) calls for keeping personal data no longer than necessary for its purpose.

| Category | Proposed period | Required behavior |
| --- | --- | --- |
| Device discovery position | Request lifetime only | No location history; redact coordinates from logs and analytics |
| Active account and identity | While account is used | On deletion request, revoke sessions and tokens immediately, cancel future hosted games, withdraw future memberships, and purge or anonymize personal records within 30 days |
| Inactive account | Review after 24 months without activity | Notify the owner and allow 30 days to return before starting deletion |
| Session and participation history | 90 days after end or cancellation | Then delete or anonymize personal history; keep only non-identifying aggregate counts if needed |
| Exact session address and map point | Until 7 days after end or cancellation | Participant access ends immediately at completion/cancellation; purge precise stored location separately from longer-lived city-level history |
| In-app notifications | 90 days | Paginated inbox; no exact address in payloads; earlier account deletion takes precedence |
| Successful delivery payloads | 7 days | Keep minimal event IDs and outcomes; minimize retained push content |
| Failed delivery diagnostics | 30 days | Redacted diagnostics for support; use bounded retries and surface permanent failure |
| Device push tokens | While installation is active | Unlink on logout, disable invalid tokens immediately, and purge after 90 days of inactivity |
| Local password credential | While the account has local login enabled | Store an Argon2id hash only; remove superseded hashes on change and purge with accepted account deletion |
| Recovery codes if adopted | Validity and cleanup to be confirmed with recovery policy | Hashes only; atomic single use; regeneration revokes earlier codes |
| Authentication session and refresh hashes | Proposed 30-day inactivity limit; minimal revoked/expired records for 30 further days | Preserve only what is required for rotation/reuse detection and security investigation |
| Application/security logs | 30 days | Exclude exact locations, passwords/hashes, recovery codes, tokens, and unnecessary profile data |
| Closed abuse reports | 180 days after closure | Restricted operator access; review open cases every 30 days and justify longer retention individually |
| Backups | Proposed rolling maximum 30 days for any copies we keep | A retention ceiling, not a promise of 30-day managed recovery; use only included history/snapshots or encrypted exports to existing private storage |
| Test fixtures and failure artifacts | Synthetic fixtures per run; diagnostics 7 days | Isolate test data and redact secrets; no production database copies in public CI |

Account deletion must cover the app database, delivery providers where supported, device associations, and derived personal data. Document vendor-managed retention that cannot be shortened through the application. Do not claim that deleting a row instantly removes older backups: track deletion requests, reapply them after any restore before serving data, and let protected backup copies expire within the stated window. A narrowly documented legal/security hold is handled separately from normal product retention.

Neon Free currently provides a limited 6-hour instant-restore window, bounded by change volume, and one manual snapshot; it does not implement the proposed 30-day recovery horizon. Validate a restore with included features. Any longer recovery scheme needs a proven free, private encrypted destination already available to the operator; its availability and automation remain open. Never use public repository files or public CI artifacts for production backups. Provider log/history windows may be shorter than our proposed maximums.

## Accepted test infrastructure

| Layer | Accepted implementation direction |
| --- | --- |
| API unit | Jest and NestJS TestingModule, with isolated adapters |
| API integration | Real PostgreSQL/PostGIS containers on Linux GitHub Actions runners; migrate empty and previous supported schemas |
| API HTTP E2E | Supertest against the real NestJS app/database; real local credential verification, with controlled push, geocoding, and social-auth boundaries |
| App unit/integration | Jest with jest-expo and React Native Testing Library; test initial snapshots and direct push-cache updates without hidden refetches |
| Android E2E | Maestro on a standard Linux runner with an emulator and an identified native build |
| iOS E2E | Maestro on a standard macOS/Xcode runner with a simulator; explicit device smoke tests for provider sign-in and push |
| Provider compatibility | Small Neon/Vercel staging smoke checks on trusted changes or release candidates; local database containers remain the fast baseline |

[Standard GitHub-hosted runner compute is free for public repositories](https://docs.github.com/en/billing/concepts/product-billing/github-actions); larger runners and storage have separate billing. Keep the three repositories public for this model and reassess before changing visibility. Extra paid CI compute and storage budget is zero. Upload only bounded, sanitized failure diagnostics within the included account storage, retain them for at most seven days, and clean them up sooner if necessary to avoid charges. Cancel superseded feature-branch runs and do not enable larger runners or paid EAS plans.

Accepted coverage gates are 80% statements/functions/lines and 75% branches for owned executable logic, with 90% branch coverage for authentication, capacity/participation, and location-authorization modules once they exist. Generated clients and declarations can be excluded explicitly; business modules cannot be excluded to manufacture a pass. Every critical scenario remains mandatory regardless of percentage.

Run API unit/integration/HTTP E2E on relevant API PRs. Run app unit/integration and Android E2E smoke on app PRs, with iOS smoke for authentication, native, navigation, and permission changes. Run the full Android/iOS regression on integrated dev and release candidates. Public fork PRs use synthetic data and no cloud, signing, or production credentials. Add real check names to branch rulesets only after those jobs exist and reliably report results.

Zero spending, test infrastructure, and coverage gates are confirmed alongside the product and provider decisions. The 18+ policy, retention periods, auth recovery/linking/session choices, durable delivery adapter, and supplemental free backup setup remain open. Confirm library versions against the scaffold without changing accepted test levels or introducing paid infrastructure.
