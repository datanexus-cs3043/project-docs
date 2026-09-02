# MedSync / CATMS - Project Documentation

Welcome to the central documentation repository for **MedSync / CATMS** (Clinical & Administrative Treatment Management System), developed by **DataNexus** for the **CS3043 Database Systems** module at the Department of Computer Science and Engineering, University of Moratuwa.

This repository serves as the single source of truth for architectural guidelines, database specifications, development standards, and project management workflows.

---

## Documentation Directory

| Document | Description |
| :--- | :--- |
| [Project Overview](project_overview.md) | Academic context, core objectives, domain background, and team structure. |
| [System Architecture](architecture.md) | Technical stack breakdown, multi-tier deployment flow, network configuration, and data flow. |
| [Database Design & Guidelines](database_design.md) | 10-step SQL script execution pipeline, schema conventions, stored procedures, triggers, and views. |
| [Development Workflow & Git Standards](development_workflow.md) | Local environment setup, Docker Compose deployment, Git branching, and Conventional Commits. |

---

## Repositories Overview

The DataNexus organization (`datanexus-cs3043`) consists of the following repositories:

- **[CATMS-Backend](https://github.com/datanexus-cs3043/CATMS-Backend)**: Spring Boot 3 application written in Java 21, utilizing Spring JDBC (`JdbcTemplate`, `NamedParameterJdbcTemplate`) for performance-oriented relational data access, integrated with MySQL 8.0 and Docker Compose.
- **[CATMS-Frontend](https://github.com/datanexus-cs3043/CATMS-Frontend)**: Modern web interface built with React, Vite, and Tailwind CSS, containerized via multi-stage Docker builds and served through Nginx.
- **[project-docs](https://github.com/datanexus-cs3043/project-docs)**: Centralized documentation repository containing architectural blueprints, database script standards, and development protocols.
- **[.github](https://github.com/datanexus-cs3043/.github)**: Organization profile and shared workspace metadata.

---

## Contribution and Governance

1. **Open Collaboration**: Any member of the DataNexus team can inspect, propose updates, or submit documentation improvements.
2. **Review Policy**: All updates to system specifications or database schemas must be submitted via Pull Request and reviewed before merging.
3. **Consistency**: All SQL scripts and Java database code must strictly align with the database guidelines documented in [`database_design.md`](database_design.md).
