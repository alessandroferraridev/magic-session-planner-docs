# Magic session planner development plan

Build a mobile application where Magic: The Gathering players worldwide can discover nearby tables, host a session, request a seat, and manage upcoming games. The backend will use NestJS and the mobile app will use React Native.

Start with a complete flow from current-location discovery to receiving a confirmed seat, opening the precise location map, and receiving in-app and push updates. Add chat and recurring sessions after that flow works reliably.

## Confirmed product choices

- Seat requests require host approval.
- Hosting uses personal accounts only, without store or organization profiles.
- At session creation, the host chooses whether to play or only provide the table. Playing is the default.
- Authentication uses Google and Apple only. NestJS owns application users, linked provider identities, and sessions; there is no local username/password or sending service.
- The mobile app targets iOS and Android through Expo.
- Hosting and discovery support locations worldwide. The interface supports English and Italian.
- Discovery starts from current device location and shows a city-only list. Approved session detail includes the exact location and map.
- Hosts can edit all session business fields and their own playing participation. Notifications include an initial in-app snapshot and push-driven UI updates, without polling.
- NestJS is hosted on Vercel and PostgreSQL/PostGIS on Neon. Email login is a possible future extension if a sending domain and delivery setup are chosen.
- Infrastructure budget is zero beyond store publication. Use free tiers and existing resources without paid upgrades or overages.
- Use Gitflow with `main` for releases, `dev` for integration and staging, and feature, fix, release, and hotfix branches.
- Implement unit, integration, and E2E tests for both API and app, starting with scaffolding and extending coverage with each feature.

## Read the plan

| Document | Purpose |
| --- | --- |
| [Product scope](./product-scope.md) | Users, screens, session rules, and MVP acceptance criteria |
| [Architecture](./architecture.md) | Technology choices, repository boundaries, and geolocation design |
| [Authentication decisions](./authentication-decisions.md) | Social-only scope, separate public identity, and remaining linking, platform, and session choices |
| [Data and API](./data-and-api.md) | Entities, endpoints, permissions, and seat allocation |
| [Development roadmap](./development-roadmap.md) | Ordered delivery steps with dependencies and completion checks |
| [Quality and release](./quality-and-release.md) | Tests, security, operations, and launch requirements |
| [Testing strategy](./testing-strategy.md) | Required unit, integration, and E2E suites, scenarios, environments, and CI gates |
| [Git workflow](./git-workflow.md) | Branches, pull requests, staging, release tags, and hotfixes across the three repositories |
| [Open decisions](./open-decisions.md) | Confirmed choices, remaining questions, and working assumptions |
| [Operating constraints](./operational-proposal.md) | Confirmed zero budget and test infrastructure; proposed retention/audience and remaining free-service dependencies |

## Workspace layout

```text
magic-session-planner-project/
  magic-session-planner-docs/         Independent documentation Git repository
  magic-session-planner-app/          Independent React Native Git repository
  magic-session-planner-api/          Independent NestJS Git repository
```

All three folders are independent Git repositories with an initial branch named `main`. The shared product and development plan is maintained in `magic-session-planner-docs`. Keep the three repositories alongside each other in the parent workspace so relative documentation links resolve. The parent workspace is not a Git repository.

Future work follows the [Git workflow](./git-workflow.md): branch from `dev`, integrate through pull requests, and promote releases to `main`. All three repositories have active GitHub rulesets and use `dev` as their default branch. CI checks and deployment pipelines remain project setup tasks.

This delivery initializes repositories and records the plan. Framework scaffolding, dependency installation, database setup, and application implementation begin in roadmap step 1.

## Initial delivery sequence

1. Resolve remaining product/authentication decisions and review operating constraints for the confirmed Vercel and Neon stack.
2. Scaffold the NestJS and React Native projects and establish local development and CI.
3. Implement authentication, profiles, venue locations, and session publishing.
4. Deliver location permission handling and nearby table discovery.
5. Implement seat requests, host approval, host edits, personal session lists, and push-driven in-app notifications.
6. Validate the full flow on real devices, deploy staging, and run a limited pilot.
