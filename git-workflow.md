# Git workflow

Use Gitflow with `main` for releases and `dev` for integration and staging. Apply the same branch roles in the API, app, and documentation repositories. API and mobile versions are independent, with explicit compatibility checks between them.

This document defines the agreed workflow. GitHub rulesets and the `dev` default branch are configured in all three repositories as described below. CI jobs and deployment automation remain project setup tasks. Existing bootstrap commits on `main` remain as the starting history.

## Branch roles

| Branch | Create from | Integrate into | Purpose |
| --- | --- | --- | --- |
| `main` | Existing bootstrap history | Not used for routine development | Released versions and their tags |
| `dev` | `main` during initial setup | Release branches | Integrate completed work and supply staging |
| `feature/<short-name>` | Current `dev` | `dev` through a pull request | Features and ordinary documentation changes |
| `fix/<short-name>` | Current `dev` | `dev` through a pull request | Non-urgent bug fixes |
| `release/<version>` | A selected `dev` commit | `main`, with release corrections returned to `dev` | Stabilize a planned release |
| `hotfix/<short-name>` | Current released `main` | `main`, `dev`, and any affected active release branch | Urgent fixes to released behavior |

Keep feature and fix branches short-lived. Use descriptive lowercase names such as `feature/nearby-tables`, `feature/docs-custom-auth-gitflow`, and `fix/session-timezone`. A production hotfix starts from the released code so unrelated development is not included.

## Pull requests and protections

- Require pull requests for changes to `main` and `dev`; block direct pushes, deletion, and force pushes on those branches after bootstrap.
- Require the relevant CI checks and resolution of review conversations. Configure required reviews to match available maintainers so a solo maintainer is not blocked waiting for an impossible second approval.
- Describe the resulting behavior, the reason for the change, validation, and any migrations or compatibility impact in each pull request.
- Use merge commits when synchronizing long-lived branches or bringing release and hotfix changes back, preserving shared ancestry. Feature and fix pull requests may be squashed into `dev`.
- Remove short-lived branches after their changes reach all intended destinations. Do not rewrite shared branch history or move published release tags.
- Use concise commit messages following the existing prefixes, such as `feat:`, `fix:`, `docs:`, and `chore:`.

## Configured GitHub rulesets

The same four active rulesets are configured in the [API repository](https://github.com/alessandroferraridev/magic-session-planner-api/settings/rules), [app repository](https://github.com/alessandroferraridev/magic-session-planner-app/settings/rules), and [documentation repository](https://github.com/alessandroferraridev/magic-session-planner-docs/settings/rules).

| Ruleset | Target | Enforced protection |
| --- | --- | --- |
| Gitflow - main | `main` | Pull requests required, review conversations resolved, deletion and force pushes blocked, merge commits only |
| Gitflow - dev | `dev` | Pull requests required, review conversations resolved, deletion and force pushes blocked, merge or squash allowed |
| Gitflow - release and hotfix | `release/*` and `hotfix/*` | Force pushes blocked; normal working updates and deletion after reintegration remain possible |
| Release tags - immutable | `v*` tags | Updates and deletion blocked; new release tags can be created |

All four rulesets have an empty bypass list, so ordinary changes by administrators also follow the rules. Creating a pull request does not grant permission to skip unresolved conversations or other enabled requirements.

Each repository has one maintainer, so the required external approval count is currently zero. Pull requests remain mandatory, stale approvals are dismissed when new reviewable changes are pushed, and unresolved review conversations block integration. Increase the approval requirement when another reviewer is available.

No CI workflows exist yet. Required status checks are therefore unset; add the real check names and require them to pass once each repository has working CI. Release candidate deployments and commit signing requirements can be configured when their supporting setup exists.

The repository default branch is `dev`. Merge commits and squash merges are enabled, rebase merges are disabled, and branch rules narrow `main` to merge commits. This preserves ancestry when promoting and synchronizing releases. Automatic branch deletion on merge is disabled; remove completed feature, fix, release, and hotfix branches explicitly after verifying reintegration. Keep `main` and `dev` permanently.

These rulesets enforce the target branch's requirements. Source branch roles in the table above remain a workflow convention; validating permitted PR source names would require an additional CI check.

## Feature and fix delivery

1. Update the local view of `origin/dev` and branch from that commit.
2. Implement a focused change and run the checks appropriate to it. Include documentation and API contract updates when behavior changes.
3. Push the working branch and open a pull request targeting `dev`.
4. Merge after the configured checks and review requirements pass.
5. CI validates the resulting `dev` commit. A passing commit is eligible for staging API deployment or an internal mobile build.

Do not merge ordinary features directly into `main`. Documentation changes also use feature branches targeting `dev`.

## Release preparation

1. Select a tested `dev` commit and create `release/<version>`.
2. Stabilize that branch with bug fixes, version updates, migration validation, and release notes. New features continue separately on `dev`.
3. Validate artifacts built from the release candidate. Keep candidate validation separate from a staging deployment that may have advanced with `dev`; the candidate must be identified by commit and artifact version.
4. Merge the release branch into `main` through a pull request and tag the resulting release commit, for example `v0.1.0`.
5. Return the released changes to `dev` through a synchronization pull request. Verify release fixes are present before deleting the release branch.
6. Deploy the tested API artifact or distribute the mobile release build according to the environment's release process.

Version tags are scoped to their repository. App `v0.2.0` does not require API `v0.2.0`; release notes record compatible API contracts and any minimum supported app version.

## Hotfix delivery

1. Create `hotfix/<short-name>` from the current released `main` commit.
2. Make the smallest complete correction, validate it, and open a pull request into `main`.
3. Merge and tag a patch release after the required checks pass.
4. Return the fix to `dev` and any affected release branch through pull requests. Verify the correction is present in each destination before deleting the hotfix branch.

A hotfix follows review and validation requirements while avoiding unrelated unreleased changes. For mobile apps, publishing still follows the store distribution process.

## Environments and artifacts

| Source | API behavior | Mobile behavior |
| --- | --- | --- |
| Feature or fix branch | CI checks; optional isolated preview | CI checks; optional developer build |
| Passing `dev` commit | Staging deployment | Internal build against staging |
| Release or hotfix candidate | Validate the identified candidate artifact | Internal testing of the identified release candidate |
| Tagged `main` release | Production release process for tested artifacts | Signed release build and store submission process |

Branch names express source control roles; they do not make an application production-ready or automatically publish a build. CI and release configuration must record the source commit, artifact version, and target environment. Keep staging and production credentials and databases separate.

## Documentation releases

In the documentation repository, `dev` contains the evolving development plan and `main` contains documentation aligned with released application behavior. Use a release branch when documentation needs coordinated stabilization for an app or API release; otherwise promote an agreed documentation milestone through a pull request to `main`. Do not tag every small text edit as an application release.

When an API contract or product behavior changes, link the corresponding app, API, and documentation pull requests. Record which documentation revision applies to a release without requiring all three repositories to share a version number.

## Initial setup checklist

- [x] Create and publish `dev` from the current bootstrap history in each repository.
- [x] Configure `dev` as the default pull request target for routine work.
- [x] Activate rulesets for `main`, `dev`, release and hotfix branches, and release tags; configure merge strategies.
- [ ] Add actual required check names once CI jobs exist.
- [ ] Configure staging, candidate validation, production delivery, mobile distribution, and tag triggers as the projects become runnable.

Keep application code, credentials, and deployment configuration changes in their respective repositories; the documentation repository records the shared process.

The workflow adapts the [original Gitflow model](https://nvie.com/posts/a-successful-git-branching-model/) using `main` and `dev`. Enforcement uses [GitHub branch protections](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches).
