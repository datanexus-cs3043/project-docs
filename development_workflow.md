# Development Workflow & Environment Setup - MedSync / CATMS

## Prerequisites

Ensure the following tools are installed on your workstation prior to setting up the project:

- **Java Development Kit (JDK)**: Version 21 LTS
- **Build Tool**: Apache Maven 3.9+
- **Node.js Environment**: Node.js 18+ and npm 9+
- **Database Server**: MySQL Server 8.0 (for local non-containerized execution)
- **Containerization**: Docker Engine 24+ and Docker Compose v2+
- **Version Control**: Git 2.40+

---

## Local Development Execution

### Option 1: Full-Stack Docker Compose (Recommended)

1. Clone all required project repositories under a common workspace directory:
   ```bash
   git clone https://github.com/datanexus-cs3043/CATMS-Backend.git
   git clone https://github.com/datanexus-cs3043/CATMS-Frontend.git
   git clone https://github.com/datanexus-cs3043/project-docs.git
   ```

2. Launch all containerized services via Docker Compose:
   ```bash
   cd CATMS-Backend
   cp .env.example .env
   docker compose up --build
   ```

3. Service endpoints:
   - **Frontend Application**: `http://localhost:5173`
   - **Spring Boot REST API**: `http://localhost:8080`
   - **MySQL Database**: `localhost:3306` (`catms_db`)

---

### Option 2: Component-Level Local Setup

For individual component development without Docker Compose, refer to the dedicated service guides:

- **Backend Development & Database Setup**: See **[CATMS-Backend README](https://github.com/datanexus-cs3043/CATMS-Backend)**.
- **Frontend Development**: See **[CATMS-Frontend README](https://github.com/datanexus-cs3043/CATMS-Frontend)**.
- **Database Schema Guidelines**: See **[Database Design & Guidelines](database_design.md)**.

---

## Team Collaboration & Git Standards

For commit conventions, branch naming schemes, issue tracking, and code review policies, refer to **[GitHub Collaboration & Git Conventions Guidelines](github_guidelines.md)**.
