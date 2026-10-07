# GitHub Collaboration & Git Conventions Draft Guidelines

This document proposes team collaboration standards, Git commit conventions, branching, issue tracking, and pull request review workflows for the **DataNexus** organization (`datanexus-cs3043`).

These guidelines remain a draft for team review and adoption across `CATMS-Backend`, `CATMS-Frontend`, and `project-docs`. They are not evidence that branch protections, required reviews, or CI checks are configured on GitHub.

---

## 1. Combined Git Commit Message Convention

Use **Conventional Commits** where helpful to keep history readable. A commit should represent one coherent, explainable change. Keep actual contributor attribution and existing meaningful messages; do not rewrite past messages just to match this draft.

### Commit Message Structure

Use a concise **Subject Header**. A summary and changes list are useful for larger changes; a small correction may need only the subject. Describe the reason, actual checks, and remaining limitations when relevant. Split work by purpose, not arbitrary file or line counts.

#### Format A: Structured Section Format (Recommended for Multi-file / Feature Commits)

```text
<type>(<scope>): <concise header summary in lowercase>

## Summary

<1-2 sentences explaining why this change was made and the high-level impact>

## Changes

- <file/component 1>: <specific change description>
- <file/component 2>: <specific change description>
- <file/component 3>: <specific change description>
```

#### Format B: Compact Bulleted Format (Recommended for Targeted Updates)

```text
<type>(<scope>): <concise header summary in lowercase>

- <bullet 1 describing change>
- <bullet 2 describing change>
- <bullet 3 describing change>
```

---

### Standard Commit Types

| Type | Description | Example Scope |
| :--- | :--- | :--- |
| `feat` | A new feature or capability | `feat(backend)`, `feat(frontend)` |
| `fix` | A bug fix or error resolution | `fix(auth)`, `fix(booking)` |
| `docs` | Documentation changes only | `docs(readme)`, `docs(api)` |
| `ci` | Continuous integration workflow changes | `ci(github-actions)` |
| `build` | Dependencies, packaging, or build configuration | `build(docker)`, `build(npm)` |
| `refactor` | Code restructuring without feature or bug behavior changes | `refactor(repo)`, `refactor(components)` |
| `style` | Code formatting, linting fixes, or whitespace adjustments | `style(linter)`, `style(format)` |
| `test` | Unit, integration, or SQL verification test updates | `test(database)`, `test(api)` |

---

### Concrete Commit Examples

Examples illustrate message structure, not changes that should be made now or claims about existing validation.

#### Example 1: Container Packaging (`build`)

```text
build(docker): configure backend and frontend application services

## Summary

Run the two application services together while the backend connects to an externally configured PostgreSQL database.

## Changes

- Backend Dockerfile: Package the FastAPI service using Python 3.11.
- Frontend Dockerfile: Build assets with Node 24 and serve them with Nginx.
- compose.yaml: Configure backend/frontend builds and host ports; no PostgreSQL service is included.
```

#### Example 2: Backend Feature Setup (`feat`)

```text
feat(backend): initialize FastAPI project structure and dependencies

## Summary

Set up core FastAPI application structure, configuration, and dependencies.

## Key Additions

- requirements.txt: Configure dependencies for FastAPI, Uvicorn, psycopg3, Pydantic, and python-dotenv.
- app/: Add main application entrypoint, core database pool, config, and initial health check route.
```

#### Example 3: Documentation Update (`docs`)

```text
docs(readme): update backend README with focused quickstart and project-docs references

- Clean up tech stack overview and repository layout tree.
- Provide concise Docker Compose and local execution instructions.
- Reference central project-docs repository for database design guidelines.
```

---

## 2. Git Branching Strategy

Use topic branches for substantial application changes and agreed integration work. Small documentation corrections can follow the team's agreed workflow without inventing an unnecessary branch. Merging a milestone into `main` does not establish that it is production-ready; consult [Implementation & Integration Status](implementation_status.md).

### Branch Protection Rules

Recommended safeguards are peer review for application changes, checks appropriate to the change, and protection against accidental direct updates to shared branches. Actual GitHub protection and review settings were not inspected for this document. Do not present these recommendations as enforced settings.

### Branch Naming Scheme

| Type | Pattern | Example |
| :--- | :--- | :--- |
| **Feature** | `feature/<issue-number>-<short-name>` | `feature/12-doctor-search-api` |
| **Bug Fix** | `bugfix/<issue-number>-<short-name>` | `bugfix/45-booking-slot-overlap` |
| **Documentation** | `docs/<short-name>` | `docs/commit-convention-guidelines` |
| **Infrastructure / CI** | `ci/<short-name>` | `ci/docker-compose-config` |

### Branch Lifecycle Workflow

1. Check the repository, current branch, working tree, and intended base. Preserve unrelated work before switching branches.
2. Update the agreed base safely; `git pull --ff-only` avoids an unexpected local merge. Choose `main` or the relevant integration branch deliberately.
3. Create a topic branch when appropriate, or continue the agreed existing branch. The naming table is a suggested convention, not a requirement to rename collaborators' branches.
4. Make focused commits, verify the intended changes, and push only the agreed branch.
5. Open a PR against the correct base. A contributor-to-integration PR and an integration-to-`main` PR have different review scopes.
6. Remove a merged branch only when no collaborator or dependent work still needs it.

---

## 3. GitHub Issue Management

Issue tracking ensures clarity on task ownership, priorities, and delivery status.

### Issue Structure Template

When creating a new GitHub Issue, provide:

1. **Title**: Imperative and clear (e.g., `Implement Doctor Search REST Endpoint`).
2. **Overview**: Brief explanation of requirement or bug report.
3. **Acceptance Criteria**: Checklist of conditions that must be satisfied.
4. **Labels**: Assign appropriate labels (`feature`, `bug`, `documentation`, `priority: high`).

### Linking Commits and PRs to Issues

Use a closing keyword only when the change actually completes the issue. In PR descriptions, GitHub interprets closing keywords when the PR targets the repository's default branch; merging into an intermediate feature branch does not close the issue through that mechanism. Closing keywords in commits take effect when those commits reach the default branch. See [GitHub's linking documentation](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/linking-a-pull-request-to-an-issue).

- `Closes #<issue-number>` (e.g., `Closes #12`)
- `Fixes #<issue-number>` (e.g., `Fixes #45`)
- `Ref #<issue-number>` (e.g., `Ref #12`) is an ordinary reference, not an official closing keyword.

Issue numbers belong to individual repositories. Use `owner/repository#number` for cross-repository references.

---

## 4. Pull Request (PR) & Code Review Guidelines

### PR Creation Checklist

Before requesting review on a Pull Request:

1. **Title**: Follow conventional commit header style (e.g., `feat(backend): add doctor search REST endpoint`).
2. **Description**: Include the purpose, changed behavior, appropriate issue references, and verification instructions/results. Separate remaining work from the PR's scope.
3. **Validation**: Run checks appropriate to the change and record their exact scope. Route registration, lint, compilation, simulator behavior, and live database verification are different forms of evidence.
4. **Reviewer Assignment**: Assign at least 1 peer reviewer from the DataNexus team.

### Code Review Standard

- **Peer Review**: Recommend at least one peer review for application changes. A review should explain the findings and unresolved limitations, not merely repeat the PR description. Follow the team's agreed process and configured repository rules.
- **Merge Strategy**: Choose merge, squash, or rebase deliberately according to the team's workflow. Preserve meaningful contributor attribution and contribution evidence; a linear history is not the only objective.
- **Scope Control**: Review and correct the submitted work without silently adding unrelated features. An approval for a bounded integration milestone is not a claim that the entire system is complete or ready for public hosting.
