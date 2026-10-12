# MedSync / CATMS — Technical Documentation

MedSync / CATMS (Clinic Appointment and Treatment Management System) is DataNexus's outpatient clinic management project for **CS3043 Database Systems**, Department of Computer Science and Engineering, University of Moratuwa.

This repository documents the system's scope, architecture, database rules, operational workflows and integration status. It is a shared reference for development, review and project submission.

## Start here

| Document | What to use it for |
| --- | --- |
| [Project Overview](project_overview.md) | Scope, five roles, the end-to-end clinic workflow and exclusions. |
| [System Architecture](architecture.md) | Frontend/API/database responsibilities, authentication, hosting and configuration. |
| [Database Design & Guidelines](database_design.md) | Relations, SQL artifacts, financial invariants and safe migration handling. |
| [Roles, Patient Portal & Reports](roles_and_reports.md) | Current action scopes, staff-managed scheduling, insurance rules and report definitions. |
| [Development Workflow](development_workflow.md) | Local setup, checks and a practical full-stack QA sequence. |
| [GitHub Collaboration](github_guidelines.md) | Integration branches, focused commits, review and release boundaries. |
| [Implementation & Integration Status](implementation_status.md) | Dated application checkpoints, completed source changes and remaining acceptance work. |

## Current documentation baseline

Updated **12 October 2026, Asia/Colombo** against backend `development` at `9e78cef` and frontend `development` at `2a941f6`. These are source checkpoints, not a certification of the deployed application.

Both application repositories use the exact **`development`** branch for current integration. Hosted `main` is a separate release baseline; do not assume it contains development changes. An application release merge requires explicit release-owner authorization. See [Implementation Status](implementation_status.md) for the database verification record and outstanding application acceptance.

## Repositories

- [CATMS-Backend](https://github.com/datanexus-cs3043/CATMS-Backend): FastAPI, PostgreSQL access and SQL artifacts.
- [CATMS-Frontend](https://github.com/datanexus-cs3043/CATMS-Frontend): React/TypeScript application.
- [project-docs](https://github.com/datanexus-cs3043/project-docs): this shared documentation.
- [.github](https://github.com/datanexus-cs3043/.github): organization profile and shared repository information.

These are independent Git repositories, not a monorepo. Local setup uses sibling application directories.

## Documentation rules

- Describe implemented source, verified runtime results and future work separately.
- Keep passwords, connection strings, tokens, patient records and private working notes out of shared documents.
- Update affected contracts and business rules alongside changes; OpenAPI describes the selected backend's request/response schema.
- Preserve existing data. SQL files are not an instruction to reset or reseed a shared database.
- Record actual implementation, review and integration responsibilities without inferring authorship from commit totals.
