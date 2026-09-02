# MedSync / CATMS - Technical Documentation

Central documentation hub for **MedSync / CATMS** (Clinical & Administrative Treatment Management System), developed by **DataNexus** for **CS3043 Database Systems** at the Department of Computer Science and Engineering, University of Moratuwa.

## Documentation Index

| Document | Focus Area |
| :--- | :--- |
| **[Project Overview](project_overview.md)** | Academic background, domain scope, key features, and stakeholder roles. |
| **[System Architecture](architecture.md)** | Multi-tier technical stack, Docker topology, network flow, and environment settings. |
| **[Database Design & Guidelines](database_design.md)** | 10-step SQL script pipeline, schema conventions, stored routines, and transaction rules. |
| **[Development Workflow](development_workflow.md)** | Local setup prerequisites, Docker Compose execution, and local build steps. |
| **[GitHub Collaboration & Git Conventions](github_guidelines.md)** | Combined commit conventions, branching strategy, issue tracking, and PR review rules. |

## Quick Repository Links

- **Backend Service**: [CATMS-Backend](https://github.com/datanexus-cs3043/CATMS-Backend)
- **Frontend Client**: [CATMS-Frontend](https://github.com/datanexus-cs3043/CATMS-Frontend)
- **Organization Profile**: [.github](https://github.com/datanexus-cs3043/.github)

## Governance Rules

1. **Consistency**: All SQL implementation scripts and Spring JDBC repositories must strictly follow [`database_design.md`](database_design.md).
2. **Git Conventions**: All repository commits and pull requests must follow [`github_guidelines.md`](github_guidelines.md).
3. **Review Process**: Any changes to system specifications or architectural rules must be submitted via Pull Request for team review.
