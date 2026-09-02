# GitHub Collaboration & Git Conventions Draft Guidelines

This document defines the team collaboration standards, Git commit conventions, branching strategy, issue tracking rules, and pull request review workflows for the **DataNexus** organization (`datanexus-cs3043`).

These guidelines are designed as a simple, professional draft for team review and adoption across `CATMS-Backend`, `CATMS-Frontend`, and `project-docs`.

---

## 1. Combined Git Commit Message Convention

To maintain a clean, readable, and structured Git history across all repositories, all commit messages must combine **Conventional Commits** syntax with a structured body layout.

### Commit Message Structure

Every commit message consists of a **Subject Header**, an optional **Summary Section**, and a mandatory or recommended **Changes/Highlights Bullet List**.

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
| `ci` | Containerization, Docker, or CI/CD workflow updates | `ci(docker)`, `ci(github-actions)` |
| `build` | Dependency or build tool configuration changes | `build(maven)`, `build(npm)` |
| `refactor` | Code restructuring without feature or bug behavior changes | `refactor(repo)`, `refactor(components)` |
| `style` | Code formatting, linting fixes, or whitespace adjustments | `style(linter)`, `style(format)` |
| `test` | Unit, integration, or SQL verification test updates | `test(database)`, `test(api)` |

---

### Concrete Commit Examples

#### Example 1: Infrastructure & Docker Setup (`ci`)
```text
ci(docker): add Dockerfile and multi-container Compose setup

## Summary

Provide containerization support for building and running the backend alongside MySQL and Frontend.

## Changes

- Dockerfile: Multi-stage build setup for packaging Spring Boot application.
- .dockerignore: Exclude target directory and IDE configs from Docker build context.
- compose.yaml: Docker Compose configuration for MySQL 8.0, Backend, and Frontend containers.
```

#### Example 2: Backend Feature Setup (`feat`)
```text
feat(backend): initialize Spring Boot project structure and dependencies

## Summary

Set up core Spring Boot application structure, configuration, and Maven dependencies.

## Key Additions

- pom.xml: Configure dependencies for Spring Boot 3, Spring Web, Spring JDBC, MySQL Connector, and Lombok.
- src/: Add main application class, application properties, and initial test setup.
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

To isolate work in progress and ensure the `main` branch remains production-ready, team members must use topic branches for all development.

### Branch Protection Rules
- Direct pushes to `main` are restricted.
- All code changes must be submitted via Pull Requests (PRs).

### Branch Naming Scheme

| Type | Pattern | Example |
| :--- | :--- | :--- |
| **Feature** | `feature/<issue-number>-<short-name>` | `feature/12-doctor-search-api` |
| **Bug Fix** | `bugfix/<issue-number>-<short-name>` | `bugfix/45-booking-slot-overlap` |
| **Documentation** | `docs/<short-name>` | `docs/commit-convention-guidelines` |
| **Infrastructure / CI** | `ci/<short-name>` | `ci/docker-compose-config` |

### Branch Lifecycle Workflow
1. Pull the latest `main` branch: `git checkout main && git pull origin main`
2. Create a new branch: `git checkout -b feature/12-doctor-search-api`
3. Commit work following the **Combined Commit Convention**.
4. Push branch to GitHub: `git push -u origin feature/12-doctor-search-api`
5. Open a Pull Request targeting `main`.
6. Delete the branch after successful merge.

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
Use official GitHub keywords in Pull Request descriptions or commit message bodies to auto-close associated issues upon merge:

- `Closes #<issue-number>` (e.g., `Closes #12`)
- `Fixes #<issue-number>` (e.g., `Fixes #45`)
- `Ref #<issue-number>` (e.g., `Ref #12` for referencing without closing)

---

## 4. Pull Request (PR) & Code Review Guidelines

### PR Creation Checklist
Before requesting review on a Pull Request:

1. **Title**: Follow conventional commit header style (e.g., `feat(backend): add doctor search REST endpoint`).
2. **Description**: Include a summary of changes, linked issue references (`Closes #12`), and testing instructions.
3. **Build Status**: Ensure code compiles cleanly without build errors.
4. **Reviewer Assignment**: Assign at least 1 peer reviewer from the DataNexus team.

### Code Review Standard
- **Peer Approval**: At least 1 approving review from a team member is required before merging into `main`.
- **Merge Strategy**: Use **Squash and Merge** or **Rebase and Merge** to maintain a linear commit history on `main`.
