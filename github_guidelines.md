# GitHub Collaboration & Git Conventions — DataNexus

This guide supports collaboration across the independent CATMS backend, frontend and documentation repositories. It records the current integration workflow and recommends review safeguards; it does not claim GitHub branch protections or required checks are configured.

## Branches and release boundary

- Both applications use the exact **`development`** branch as the shared integration baseline.
- `main` is the release/hosted baseline, not the target for ordinary unfinished integration.
- Topic branches are useful for isolated work and PR review; existing collaborator branch names need not be renamed.
- Documentation follows its own repository workflow. A documentation branch or commit does not release either application.
- **Do not merge application development into main until Shashika explicitly authorizes the release; Nayanajith may perform the release merge himself.** Routine scoped integration work is not implicit main-release approval.

| Work type | Suggested topic name | Application PR target |
| --- | --- | --- |
| Feature | `feature/<short-name>` | `development` |
| Fix | `bugfix/<short-name>` | `development` |
| Infrastructure | `ci/<short-name>` | `development`, unless a separately authorized release/configuration task |
| Documentation | `docs/<short-name>` | Agreed base in the documentation repository |

Before Git operations, verify repository, branch, working tree and intended upstream. Use safe fast-forward pulls when possible. Stop on overlapping local changes or divergence; do not hard-reset, force-push or rewrite shared history to make integration easier.

Before pushing, inspect the intended commits, remote and destination branch. A local commit, pushed branch, reviewed PR, merged change and deployed feature are different milestones.

## Focused commits

A commit should represent one coherent, explainable change. Separate unrelated authentication, clinical, billing, reporting and documentation work; do not split only to manufacture activity.

Use concise conventional subjects where useful:

```text
fix(appointments): preserve cancelled slot availability
feat(patients): add optional staff-managed portal access
docs(reports): clarify receipts and insurance approval totals
```

For a larger change, add the reason, affected contracts and actual verification:

```text
<type>(<scope>): <concise behavior change>

Why:
<problem and intended outcome>

Changes:
- <specific implementation or documentation change>

Verification:
- <check actually run and its environment>
- <remaining runtime or migration limitation>
```

Common types: `feat`, `fix`, `docs`, `refactor`, `build`, `ci`, `style`, `test`. Use `test` for actual verification-artifact changes, not as a substitute for running checks. Keep meaningful existing history and actual contributor attribution.

## Issues and handoff

A useful issue/handoff includes:

1. Clear problem, affected role/workflow and current source/environment.
2. Reproduction or contract evidence with sanitized request/response details.
3. Expected behavior, business-rule decision and acceptance criteria.
4. Assigned implementer/reviewer once agreed, related repository/PR references and dependencies.
5. Database/configuration impact and anything requiring separate approval.

Issue numbers are repository-specific. Use qualified references for cross-repository work. Reference an issue while work is partial; use closing language only when its acceptance criteria are actually complete.

For a full-stack change, state both frontend/backend commits and any required migration. Do not close a workflow solely because its button renders or its route appears in OpenAPI.

## PR and review checklist

- Confirm the correct base, focused diff and preservation of unrelated teammate work.
- Explain changed behavior, request/response contracts, roles/scopes, financial meaning and failure states.
- Review transactions, captured historical prices, mutation retries and migration ordering where relevant.
- Run appropriate checks and state what they prove. Static builds, isolated handlers, PostgreSQL concurrency and browser journeys are separate evidence.
- Document remaining gaps honestly, especially an unapplied migration or unverified deployed version.
- Request a peer review for application changes and follow the configured repository rules. These are recommended safeguards, not a statement that reviews are enforced.
- Preserve meaningful attribution when selecting merge/squash/rebase. A merge commit alone does not identify the implementer of every feature.
- Delete merged branches only when no collaborator or dependent work still requires them.

## Shared documentation and contribution records

Update the relevant document when behavior changes; [Implementation Status](implementation_status.md) owns the dated checkpoint and acceptance handoff. Avoid parallel contradictory status lists.

Contribution descriptions should identify the area, actual implementation/review/integration work and supporting commits/PRs. Do not infer percentages, sole feature ownership or equal participation from commit/line counts. Keep private preparation notes, credentials, real records and local adapters outside submission/shared history.

Recommended contribution entry:

```text
Contributor:
Area and responsibility:
Implementation evidence (repository + commit/PR):
Review/integration evidence:
Verification performed:
Remaining handoff:
```

## Release checklist

Before an authorized application release:

1. Finish the agreed role-scoped acceptance journeys and reconcile the ER/SRS with the candidate.
2. Validate required migrations on PostgreSQL, separately approve shared application and record actual results.
3. Review secrets/cookie/CORS/API configuration and hosted build/startup settings.
4. Record the paired application SHAs, schema version and remaining limitations.
5. Obtain explicit main-release authorization, then verify the resulting deployment and database health.

A successful build or deployment job does not replace acceptance. Current release gates are maintained in [Implementation Status](implementation_status.md).
