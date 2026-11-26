# Budget retention audience and test infrastructure proposal

The product owner has selected Vercel for NestJS, Neon for PostgreSQL/PostGIS, and Resend for email. The budget, retention periods, audience age policy, delivery adapter, and test infrastructure settings below are proposals for confirmation. No paid account, resource, or spending limit has been provisioned by this documentation change.

## Proposed monthly budget

Plan for an initial pilot of roughly 100 to 500 monthly active players, one developer, English/Italian content, and modest session traffic. These are sizing assumptions rather than verified capacity limits or a restriction on worldwide use. Target 60 to 80 USD per month and review any forecast above 100 USD. Figures exclude taxes, currency conversion, developer time, and store registration fees.

| Item | Monthly planning allowance | Basis |
| --- | --- | --- |
| Vercel Pro, one developer | 20 USD | Published plan price; platform usage is metered against applicable allowances |
| Extra function and durable delivery usage | 10 USD reserve | Estimate, not an included service guarantee; validate the chosen Workflow/Queue adapter and its pricing |
| Neon production and staging, storage and recovery history | 15 USD reserve | Usage estimate for intermittent small workloads; cap autoscaling and monitor actual CU-hours |
| Resend | 0 USD pilot, or 20 USD Pro | Free: 3,000 emails/month and 100/day; Pro: 50,000/month with no daily limit |
| Push transport | 0 USD provider sending fee proposed | Expo push service candidate; application execution and delivery state still use Vercel/Neon resources |
| Mapping and geocoding | 10 USD reserve | Provider not selected; validate approved-detail map costs, worldwide city/address coverage, quotas, and attribution |
| Domain, backup storage, and monitoring | 5 USD reserve | Planning allowance, including annual costs amortized monthly; re-estimate against chosen services |
| Standard CI runner compute | 0 USD baseline | The three repositories are public; use standard GitHub-hosted runners and account for storage separately |
| Total | 60 USD with free email; 80 USD with Resend Pro | Proposal arithmetic, not a fixed-price hosting contract |

Price checks dated 26 November 2026: [Vercel pricing](https://vercel.com/pricing), [Neon pricing](https://neon.com/pricing), [Resend pricing](https://resend.com/pricing), [Expo push fees](https://docs.expo.dev/push-notifications/faq/), and [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions).

Neon Launch lists 0.106 USD per CU-hour and 0.35 USD per GB-month; the 15 USD allowance assumes intermittent use, not an always-active database. A continuously active illustrative 0.25 CU compute uses about 182.5 CU-hours in a 730-hour month, or 19.35 USD before storage and history. Do not poll notifications or run minute-by-minute idle database scans. Measure production and staging separately and revise the allowance if activity prevents scale-to-zero. Neon supports the required [PostGIS extension](https://neon.com/docs/extensions/postgis).

Use spend alerts around 80 and 100 USD where supported, conservative compute limits, and explicit review before enabling paid overages or larger tiers. A budget alert is not a universal hard cap. Do not stop critical writes or authentication automatically as a budget-control shortcut. The Vercel Pro choice is a proposal; compare the final needed features and terms before purchase, rather than assuming that a paid plan is required merely to avoid polling.

For geocoding, [Geoapify's free tier](https://www.geoapify.com/pricing/) is a candidate with 3,000 credits/day and required attribution. This does not select the detail-map provider or guarantee all map usage is free. City lookup and host venue entry must remain separate from a manual discovery destination.

Store distribution is additional: [Apple Developer Program](https://developer.apple.com/programs/enroll/) lists 99 USD/year, with regional pricing, and [Google Play registration](https://support.google.com/googleplay/android-developer/answer/6112435?hl=en) lists 25 USD once. Existing memberships can cover these costs. Initial app builds can use CI runners; a paid EAS subscription is not assumed in this budget.

## Proposed deployment arrangement

- Use separate Vercel projects for staging and production, and separately credentialed Neon resources. Place the API compute near its database; an initial European location is an infrastructure proposal, not a limit on where users can host sessions.
- Keep application authentication custom in NestJS; selecting Neon does not select Neon's managed authentication product.
- Use event-triggered durable notification work rather than polling the notification table. Prototype the recovery boundary between database commit and delivery with the selected Vercel Workflow or Queue SDK before finalizing it.
- Load an initial notification snapshot and patch foreground UI state from safe push payloads. Reconcile once on app entry/foreground return or explicit refresh to recover missed events; do not refetch for every push.
- Configure session-end work for the relevant session version and revalidate after edits. Retention cleanup is a bounded daily maintenance task, not a notification refresh loop.
- Keep staging pushes and emails confined to test recipients. Preview deployments use synthetic data and never inherit production credentials.

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
| Magic-link challenge | Proposed 10-minute validity; purge within 24 hours | Store a hash; consume once; never keep a usable expired credential |
| Authentication session and refresh hashes | Proposed 30-day inactivity limit; minimal revoked/expired records for 30 further days | Preserve only what is required for rotation/reuse detection and security investigation |
| Application/security logs | 30 days | Exclude exact locations, tokens, email links, and unnecessary profile data |
| Closed abuse reports | 180 days after closure | Restricted operator access; review open cases every 30 days and justify longer retention individually |
| Backups | Rolling maximum 30 days | Configure provider history/snapshots or encrypted exports to an explicit expiry and restrict restore access |
| Test fixtures and failure artifacts | Synthetic fixtures per run; diagnostics 7 days | Isolate test data and redact secrets; no production database copies in public CI |

Account deletion must cover the app database, delivery providers where supported, device associations, and derived personal data. Document vendor-managed retention that cannot be shortened through the application. Do not claim that deleting a row instantly removes older backups: track deletion requests, reapply them after any restore before serving data, and let protected backup copies expire within the stated window. A narrowly documented legal/security hold is handled separately from normal product retention.

## Proposed test infrastructure

| Layer | Proposed implementation |
| --- | --- |
| API unit | Jest and NestJS TestingModule, with isolated adapters |
| API integration | Real PostgreSQL/PostGIS containers on Linux GitHub Actions runners; migrate empty and previous supported schemas |
| API HTTP E2E | Supertest against the real NestJS app and database; controlled external email, push, geocoding, and social-auth boundaries |
| App unit/integration | Jest with jest-expo and React Native Testing Library; test initial snapshots and direct push-cache updates without hidden refetches |
| Android E2E | Maestro on a standard Linux runner with an emulator and an identified native build |
| iOS E2E | Maestro on a standard macOS/Xcode runner with a simulator; explicit device smoke tests for provider sign-in and push |
| Provider compatibility | Small controlled Neon/Vercel/Resend staging smoke checks on trusted changes or release candidates; local database containers remain the fast baseline |

[Standard GitHub-hosted runner compute is free for public repositories](https://docs.github.com/en/billing/concepts/product-billing/github-actions); larger runners and storage have separate billing. Reassess if repository visibility changes. Proposed extra paid CI compute budget is zero initially, with no larger runners or paid EAS build plan enabled automatically. Cap artifact retention and cancel superseded feature-branch runs.

Proposed coverage gates are 80% statements/functions/lines and 75% branches for owned executable logic, with 90% branch coverage for authentication, capacity/participation, and location-authorization modules once they exist. Generated clients and declarations can be excluded explicitly; business modules cannot be excluded to manufacture a pass. Every critical scenario remains mandatory regardless of percentage.

Run API unit/integration/HTTP E2E on relevant API PRs. Run app unit/integration and Android E2E smoke on app PRs, with iOS smoke for authentication, native, navigation, and permission changes. Run the full Android/iOS regression on integrated dev and release candidates. Public fork PRs use synthetic data and no cloud, email, signing, or production credentials. Add real check names to branch rulesets only after those jobs exist and reliably report results.

Confirm budget, the 18+ policy, retention periods, delivery adapter, and coverage thresholds before treating them as implementation constraints. Provider selection, personal profiles, current-position discovery, approval-gated exact location, host editing, and no-polling push-driven UI updates are already confirmed.
