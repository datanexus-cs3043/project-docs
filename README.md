# MedSync / CATMS - Technical Documentation

Central documentation hub for **MedSync / CATMS** (Clinic Appointment and Treatment Management System), developed by **DataNexus** for **CS3043 Database Systems** at the Department of Computer Science and Engineering, University of Moratuwa.

## Documentation Index

| Document | Focus Area |
| :--- | :--- |
| **[Project Overview](project_overview.md)** | Academic background, intended domain scope, and the five stakeholder roles. |
| **[System Architecture](architecture.md)** | Current technical stack, Docker topology, request flow, and environment settings. |
| **[Database Design & Guidelines](database_design.md)** | Numbered SQL file organization, current schema boundaries, and transaction requirements. |
| **[Development Workflow](development_workflow.md)** | Local setup, Docker Compose execution, and appropriately scoped verification. |
| **[GitHub Collaboration & Git Conventions](github_guidelines.md)** | Draft commit conventions, branching, issue tracking, and PR review guidance. |
| **[Implementation & Integration Status](implementation_status.md)** | Dated source checkpoint, frontend/backend contract gaps, and remaining verification. |
| **[Roles, Patient Self-Service & Report Output](roles_and_reports.md)** | Five-role scope, proposed abilities versus actual permissions, deferred patient work and PDF view/download requirements. |

## Quick Repository Links

- **Backend Service**: [CATMS-Backend](https://github.com/datanexus-cs3043/CATMS-Backend)
- **Frontend Client**: [CATMS-Frontend](https://github.com/datanexus-cs3043/CATMS-Frontend)
- **Organization Profile**: [.github](https://github.com/datanexus-cs3043/.github)

These repositories and `project-docs` have separate Git histories. The common parent folder is a workspace, not a monorepo.

## Governance Rules

1. **Consistency**: Keep specifications, repository SQL, and API behavior aligned. Use [Database Design](database_design.md) to distinguish the current schema from intended changes; do not treat a proposal as an approved or deployed change.
2. **Git Conventions**: Use [GitHub Guidelines](github_guidelines.md) as a draft for team agreement. It does not establish which branch protections or required checks are configured on GitHub.
3. **Review Process**: Submit specification and architecture changes for team review. Record their rationale and any effect on the ER model, API contract, and evaluation evidence.
4. **Evidence**: Intended scope is not a completion claim. Browser-local demo results, registered routes, and successful static checks are not substitutes for live database and end-to-end verification. Refer to the dated [implementation status](implementation_status.md).
