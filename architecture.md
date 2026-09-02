# System Architecture - MedSync / CATMS

## Architectural Overview

MedSync / CATMS follows a classic multi-tier client-server architecture composed of a React frontend client, a Spring Boot REST API backend, and a MySQL relational database engine. Dockerfiles and a Docker Compose configuration exist for the frontend, backend, and MySQL services. Individual images have been built and run, but the complete clean three-service Compose workflow is still being verified.

```mermaid
graph TD
    Client[Web Browser Client] -->|HTTP / REST API| Frontend[CATMS-Frontend: Nginx Container / Port 5173]
    Frontend -->|API Requests| Backend[CATMS-Backend: Spring Boot Container / Port 8080]
    Backend -->|Spring JDBC / Connection Pool| Database[(MySQL 8.0 Database / Port 3306 - catms_db)]
```

---

## Component Breakdown

### 1. Presentation Layer (`CATMS-Frontend`)

- **Framework**: React 19.2.8 with Vite 8.2.2.
- **Styling**: Tailwind CSS with responsive layout components.
- **State & Routing**: Component-level React hooks (`useState`, `useMemo`), single-page application structure.
- **Production Build**: Multi-stage Docker build using `node:20-alpine` for asset compilation and `nginx:alpine` for static hosting.
- **Port Mapping**: Container port 80 mapped to host port 5173.

#### Key UI Modules:
- **Navbar & Navigation**: Sticky header with brand logo, search navigation, and user authentication actions.
- **Hero & Doctor Search Bar**: Live search filtering by doctor name, specialty, or hospital affiliation.
- **Specialty Catalog**: Categorized medical specialties (Cardiology, Neurology, Pediatrics, Dermatology, Dentistry, Ophthalmology).
- **Appointment Channeling List**: Real-time listing of available doctors with rating badges, hospital affiliations, and time slots.
- **Booking Modal**: Channel confirmation dialog capturing patient information and issuing appointment confirmation.

---

### 2. Application & API Layer (`CATMS-Backend`)

- **Runtime**: Java 21 LTS.
- **Framework**: Spring Boot 3 (`spring-boot-starter-web`).
- **Data Access Layer**: Spring JDBC (`JdbcTemplate`, `NamedParameterJdbcTemplate`).
  - Chosen over heavy ORM frameworks to maintain precise control over SQL queries, stored procedure calls, transactions, and execution optimization required for the CS3043 Database Systems module.
- **Dependencies**:
  - `spring-boot-starter-web`: REST API endpoints and HTTP request handlers.
  - `spring-boot-starter-jdbc`: Relational database connection pooling and SQL execution.
  - `mysql-connector-j`: Official MySQL JDBC driver.
  - `lombok`: Boilerplate reduction for data models and DTOs.
  - `spring-boot-starter-test`: Unit and integration testing utilities.
- **Build Tool**: Apache Maven (`pom.xml`).
- **Port Mapping**: Container port 8080 mapped to host port 8080.

---

### 3. Data Storage Layer (`catms_db`)

- **Database Engine**: MySQL 8.0.
- **Database Name**: `catms_db`.
- **Port**: 3306.
- **Initialization & Schema Design**: The database structure is organized into a 10-step sequential SQL script pipeline executed from `CATMS-Backend/database/`.
  - For the complete script execution pipeline, table dependencies, and schema conventions, refer to **[Database Design & Guidelines](database_design.md)**.

---

## Containerization & DevOps Setup

The entire solution is orchestrated using Docker Compose (`compose.yaml` in `CATMS-Backend`).

### Network Topology
- **Container Network**: Docker Compose provides the default project network. Services communicate using Compose service names such as `mysql` and `backend`.
- **Health Checks**: MySQL container includes `mysqladmin ping` health check to ensure database readiness before backend startup.
- **Persistence**: Named Docker volume `mysql_data` attached to `/var/lib/mysql` to ensure persistent storage across container restarts.

---

## Environment Variables

| Variable | Description | Default Value |
| :--- | :--- | :--- |
| `DB_HOST` | MySQL hostname | `localhost` locally, `mysql` in Compose |
| `DB_PORT` | MySQL port | `3306` |
| `DB_NAME` | Database name | `catms_db` |
| `DB_USER` | Database username | `root` |
| `DB_PASSWORD` | Database password | Local example value |
| `MYSQL_ROOT_PASSWORD` | MySQL container root password | Local example value |
| `VITE_API_BASE_URL` | Planned frontend API base URL | `http://localhost:8080/api` |