# Magic session planner development plan

Build a mobile application where Magic: The Gathering players worldwide can discover nearby tables, host a session, request a seat, and manage upcoming games. The backend will use NestJS and the mobile app will use React Native.

Start with a complete flow from finding a nearby table to receiving a confirmed seat. Add maps, chat, and advanced store features after that flow works reliably.

## Confirmed product choices

- Seat requests require host approval.
- At session creation, the host chooses whether to play or only provide the table. Playing is the default.
- Authentication is managed by the NestJS backend, with email magic links, Google, and Apple sign-in. The backend owns application users, sessions, and tokens.
- The mobile app targets iOS and Android through Expo.
- Hosting and discovery support locations worldwide. The interface supports English and Italian.
- Use Gitflow with `main` for releases, `dev` for integration and staging, and feature, fix, release, and hotfix branches.
- Implement unit, integration, and E2E tests for both API and app, starting with scaffolding and extending coverage with each feature.

## Read the plan

| Document | Purpose |
| --- | --- |
| [Product scope](./product-scope.md) | Users, screens, session rules, and MVP acceptance criteria |
| [Architecture](./architecture.md) | Technology choices, repository boundaries, and geolocation design |
| [Data and API](./data-and-api.md) | Entities, endpoints, permissions, and seat allocation |
| [Development roadmap](./development-roadmap.md) | Ordered delivery steps with dependencies and completion checks |
| [Quality and release](./quality-and-release.md) | Tests, security, operations, and launch requirements |
| [Testing strategy](./testing-strategy.md) | Required unit, integration, and E2E suites, scenarios, environments, and CI gates |
| [Git workflow](./git-workflow.md) | Branches, pull requests, staging, release tags, and hotfixes across the three repositories |
| [Open decisions](./open-decisions.md) | Confirmed choices, remaining questions, and working assumptions |

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

1. Resolve the remaining MVP product decisions, define account linking, and choose email delivery and hosting services.
2. Scaffold the NestJS and React Native projects and establish local development and CI.
3. Implement authentication, profiles, venue locations, and session publishing.
4. Deliver location permission handling and nearby table discovery.
5. Implement seat requests, host approval, cancellation, and personal session lists.
6. Validate the full flow on real devices, deploy staging, and run a limited pilot.
