# Documentation decision audit

Reviewed on 26 November 2026 against the owner's latest decisions and zero infrastructure budget. All original shared-plan documents and both application READMEs were reviewed. Authentication implementation, retention, audience, free event delivery and backup strategy are accepted. **Only initial formats remain open.** Implementation and provisioning are still future work; this audit does not mark them delivered.

## Review coverage

| Document | Result |
| --- | --- |
| [README](./README.md) | Index/sequence reflect accepted policies and one open format decision |
| [Product scope](./product-scope.md) | 18+, homes, authenticated browsing, foreground-only location, terminal history and host edit rules accepted |
| [Architecture](./architecture.md) | Concrete ORM, app libraries, auth, mapping, push and durable execution choices; supported versions pinned at scaffold |
| [Authentication](./authentication-decisions.md) | Linking, last-provider rule, recovery limitations, both platforms, identifiers, tokens, keys and immediate revocation defined |
| [Data/API](./data-and-api.md) | Auth/operation/format contracts and entities, private locations, body-based discovery and completion generation aligned |
| [Roadmap](./development-roadmap.md) | Policy votes replaced by implementation acceptance tasks; formats alone block catalogue-specific work |
| [Quality/release](./quality-and-release.md) | Retention, manual backups, private support/operator duties and free-plan gates aligned |
| [Tests](./testing-strategy.md) | Unit/integration/API E2E/native E2E retained; auth, retention, durable command, no-polling and lifecycle coverage extended |
| [Git workflow](./git-workflow.md) | Existing Gitflow/rulesets retained; CI/deployment configuration accurately remain unimplemented |
| [Operating policy](./operational-proposal.md) | Zero-cost services, non-commercial pilot, 18+, actual retention deadlines and limited manual recovery selected |
| [Open decisions](./open-decisions.md) | Exactly one open row: initial formats; Commander is an example/candidate only |
| [API README](../magic-session-planner-api/README.md) / [app README](../magic-session-planner-app/README.md) | Obsolete magic-link/provider-undecided descriptions removed; dev workflow and canonical policy links added |

[Delivery and recovery](./delivery-and-recovery.md) is the new implementation contract for the notification durability gap found in the audit.

## Inconsistencies resolved

1. Removed pending approval language for auth, retention, audience and free services. Confirmed foreground-only location and authenticated browsing rather than leaving them as assumptions.
2. Removed implicit Commander-only scope from product rules/examples. Catalogue storage is fixed, but initial values await the owner's format decision.
3. Selected Google SDK on both platforms, Apple native iOS and browser Android, with provider-hosted HTTPS callbacks and proof-bound app return. Removing magic links does not remove social callback requirements.
4. Defined fresh explicit linking, last-provider protection, no account merge, immediate server-side revocation, multi-device sessions and 15-minute/30-day/90-day limits. Separated public names from identities and provider recovery from app support.
5. Selected free Geoapify/MapLibre and Expo push, attribution and quota behavior. Private-home and public-space categories have identical approval-gated exact-location disclosure. Only canonical locality data drives discovery.
6. Removed approximate-distance labels from the roadmap because public location is city-only. Search coordinates and host address lookup move to POST bodies with body logging disabled so platform URL logs do not capture them. Provider location processing must be disclosed separately.
7. Defined durable workflow acceptance before business mutation, transactional inbox/delivery state, asynchronous pending behavior, bounded receipt/retry work and targeted operator recovery. No polling loop fills a database-commit gap.
8. Separated completionGeneration from aggregate version. Otherwise a later seat approval could invalidate the only scheduled completion. Protected access uses server end time even when the job is late.
9. Distinguished reusable owner-saved venue addresses from session snapshots. All precise session revisions expire seven days after end/cancel; saved locations remain owner-controlled until deletion/inactivity rules apply.
10. Made cancelled/completed history terminal and retained host editing of all business fields in active draft/published sessions with capacity/ownership constraints. A new game uses a new draft. Ownership transfer and individual player removal by hosts are deferred scope, not pending MVP design.
11. Replaced an unverified 30-day managed backup expectation with limited included Neon restore plus weekly/pre-migration encrypted manual local exports, deletion-ledger replay and explicit failure limits. Thirty days is a maximum copy retention, not guaranteed recovery history.
12. Defined 18+ self-declaration, homes in the pilot, personal/non-commercial free use, private operator CLI and report response target. Actual support contact/published policies are release deliverables; no additional audience vote is required.

## Outstanding work, without additional product questions

| Stage | Task and evidence |
| --- | --- |
| Scaffold | Pin supported versions/lockfiles; create projects, migrations, unit/integration/E2E suites and real CI status checks |
| Provider setup | Register chosen app/service identifiers, configure Google/Apple clients, signing/APNs/FCM, stable Vercel hostnames, isolated Frankfurt Neon projects and Geoapify keys |
| Technical validation | Prove beta Workflow/Nest consumer deployment, command/commit crash recovery, quota limits, social callbacks on both platforms and safe city resolution/map attribution |
| Operations | Implement daily bounded cleanup, manual encrypted export/ledger tooling, operator CLI, usage review and a private restore drill with no production data in public CI |
| Pilot/release | Publish bilingual privacy/community/support and deletion-request pages on free GitHub Pages, configure a private support contact, verify both stores' disclosure/build requirements and record real-device/provider checks |
| Format decision | Choose initial supported formats before publishing/filter catalogue fixtures are finalized; independent scaffold/auth work can proceed |

No service account, billing change, deployment, app scaffold, CI suite or backup job is created by these documentation decisions. Any technical gate failure blocks the affected milestone until fixed within the accepted zero budget; it does not silently reopen product choices or authorize paid services.
