# Development Workflow & Git Standards - MedSync / CATMS

## Prerequisites

Ensure the following tools are installed on your workstation prior to setting up the project:

- **Java Development Kit (JDK)**: Version 21 LTS
- **Build Tool**: Apache Maven 3.9+
- **Node.js Environment**: Node.js 18+ and npm 9+
- **Database Server**: MySQL Server 8.0 (for non-Docker execution)
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

2. Navigate to `CATMS-Backend` and initialize the environment variables:
   ```bash
   cd CATMS-Backend
   cp .env.example .env
   ```

3. Build and launch all services:
   ```bash
   docker compose up --build
   ```

4. Service endpoints:
   - **Frontend Application**: `http://localhost:5173`
   - **Spring Boot REST API**: `http://localhost:8080`
   - **MySQL Database**: `localhost:3306` (`catms_db`)

---

### Option 2: Manual Local Setup

#### Backend Setup:
1. Start local MySQL 8.0 instance.
2. Execute SQL scripts in `CATMS-Backend/database/` sequentially from `01_database.sql` to `10_tests.sql`.
3. Configure `src/main/resources/application.properties` with database connection parameters.
4. Run application:
   ```bash
   cd CATMS-Backend
   mvn spring-boot:run
   ```

#### Frontend Setup:
1. Navigate to `CATMS-Frontend`.
2. Install npm packages:
   ```bash
   cd CATMS-Frontend
   npm install
   ```
3. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
4. Start development server:
   ```bash
   npm run dev
   ```

---

## Git Branching Strategy

To maintain repository stability across team members, the following branching model is enforced:

- **`main`**: Production-ready, stable codebase. Direct commits to `main` are restricted.
- **`feature/<short-description>`**: Feature development branches (e.g., `feature/patient-booking-api`, `feature/doctor-search-ui`).
- **`bugfix/<issue-description>`**: Bug resolution branches (e.g., `bugfix/appointment-slot-overlap`).
- **`docs/<topic>`**: Documentation updates (e.g., `docs/schema-architecture`).

---

## Conventional Commits Specification

All commit messages across `CATMS-Backend`, `CATMS-Frontend`, and `project-docs` must adhere to the Conventional Commits specification:

```text
<type>(<scope>): <short summary>

[optional body describing detailed changes]
```

### Commit Types:
- `feat`: A new feature added to backend or frontend.
- `fix`: A bug fix.
- `docs`: Documentation updates only.
- `style`: Formatting, missing semi-colons, white-space changes.
- `refactor`: Code changes that neither fix a bug nor add a feature.
- `test`: Adding or correcting unit/integration tests.
- `ci`: Changes to CI/CD workflows, Dockerfiles, or compose files.
- `build`: Changes that affect the build system or Maven/npm dependencies.

### Commit Example:
```text
feat(backend): implement transactional appointment booking procedure call

- Add call to 07_procedures.sql channeling booking logic in Spring JDBC repository.
- Handle constraint violation exceptions and return structured HTTP 400 response.
```

---

## Code Review & Pull Request Guidelines

1. **Pull Request Creation**: Open PRs targeting `main` with clear titles, descriptions, and issue numbers.
2. **Review Requirement**: At least one peer review approval from a team member is required before merging.
3. **Build Validation**: Code must compile without errors, pass all SQL test scripts (`10_tests.sql`), and pass frontend linter (`npm run lint`).
