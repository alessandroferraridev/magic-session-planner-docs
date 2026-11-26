# Magic session planner

Development plan for a mobile app to organize in-person Magic: The Gathering sessions worldwide. The API uses NestJS; the iOS/Android app uses React Native with Expo.

The repositories currently contain the development plan and bootstrap files. Application scaffolding, provider registrations, CI suites and deployment are implementation work in the roadmap. The only open product choice is the initial format catalogue.

## Documentation by topic

Each document owns the rules for its topic. Other documents link to those rules rather than maintain separate decision summaries.

| Topic | Document | Owns |
| --- | --- | --- |
| Product | [Product scope](./product-scope.md) | Audience, hosting/joining, post-start leaving, blocking, deletion, demo experience and MVP acceptance |
| Authentication | [Authentication](./authentication.md) | Google/Apple flows, onboarding, identity linking, tokens, callbacks and revocation |
| Architecture | [Architecture](./architecture.md) | Stack, modules, repository boundaries, geographic search and localization design |
| Data and contracts | [Data and API](./data-and-api.md) | Entities, endpoints, immediate safety transactions, numeric admission limits and seat allocation |
| Notifications and jobs | [Delivery and recovery](./delivery-and-recovery.md) | Command outcomes/idempotency, snapshot/event ordering, push, lifecycle jobs and recovery without polling |
| Operations | [Operations](./operations.md) | Free infrastructure, environments, quotas, support operations, retention and backup/restore duties |
| Tests | [Testing strategy](./testing-strategy.md) | Unit/integration/E2E suites, test tools/runners, coverage, fixtures and CI gates |
| Release readiness | [Quality and release](./quality-and-release.md) | Performance/privacy checks, isolated store-review demo, release evidence and rollback readiness |
| Development | [Development roadmap](./development-roadmap.md) | Ordered implementation steps, dependencies and completion checks |
| Git | [Git workflow](./git-workflow.md) | Branch roles, pull requests, GitHub protections, staging and release promotion |
| Open question | [Open decisions](./open-decisions.md) | Initial formats only |

Start with product scope, then architecture and the roadmap. Consult the dedicated specification when implementing a feature and update that document when its requirements change.

## Workspace layout

```text
magic-session-planner-project/
  magic-session-planner-docs/         Shared specifications and development plan
  magic-session-planner-app/          React Native app and its tests/builds
  magic-session-planner-api/          NestJS API, schema, migrations and API tests
```

These are independent Git repositories. Keep them beside each other so relative cross-repository documentation links resolve. The parent workspace is not a Git repository.

All three repositories use protected `dev` as the default branch and `main` for release milestones. Routine work starts from `dev` and integrates through pull requests under the Git workflow. Existing GitHub rulesets are configured; real CI checks and delivery pipelines are roadmap tasks.
