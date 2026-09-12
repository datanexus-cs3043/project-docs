# System Architecture - MedSync / CATMS

## Architectural Overview

MedSync / CATMS follows a multi-tier client-server architecture composed of a React frontend client, a Python FastAPI REST API backend, and a PostgreSQL relational database engine hosted on Neon (with local container support for offline development).

```mermaid
graph TD
    Client[Web Browser Client] -->|HTTP / REST API| Frontend[CATMS-Frontend: React / Vite / Nginx / Port 5173]
    Frontend -->|API Requests| Backend[CATMS-Backend: Python FastAPI / Port 8000]
    Backend -->|SQLAlchemy / asyncpg Connection Pool| Database[(PostgreSQL Database - Neon Cloud / Port 5432)]
```

---

## Component Breakdown

### 1. Presentation Layer (`CATMS-Frontend`)

- **Framework**: React 19 with Vite.
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

- **Runtime**: Python 3.11+.
- **Framework**: FastAPI (`uvicorn` ASGI server).
- **Data Access Layer**: Direct SQL queries / SQLAlchemy Core / asyncpg / psycopg.
  - Chosen to provide lightweight, high-performance asynchronous REST endpoints while retaining explicit control over SQL queries, stored routines, transactions, and concurrency required for the CS3043 Database Systems module.
- **Dependencies**:
  - `fastapi`: High-performance async API framework.
  - `uvicorn`: ASGI web server implementation.
  - `pydantic`: Request validation and data serialization models.
  - `psycopg2-binary` / `asyncpg`: PostgreSQL database driver for Python.
  - `python-dotenv`: Environment variable management.
- **Port Mapping**: Container/service port 8000 mapped to host port 8000.

---

### 3. Data Storage Layer (`PostgreSQL` / `Neon`)

- **Database Engine**: PostgreSQL 16+.
- **Cloud Hosting**: Neon Serverless PostgreSQL (`neon.tech`) for centralized team access.
- **Initialization & Schema Design**: The database structure is organized into a sequential SQL script pipeline (tables, constraints, indexes, views, functions, procedures, triggers, seed data, tests).
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
 
| Variable | Description | Default / Example Value |
| :--- | :--- | :--- |
| `DATABASE_URL` | PostgreSQL connection URL (Neon / local) | `postgresql://user:password@ep-xyz.neon.tech/neondb?sslmode=require` |
| `DB_HOST` | Database host | `localhost` or Neon cloud endpoint |
| `DB_PORT` | PostgreSQL port | `5432` |
| `DB_NAME` | Database name | `catms_db` or `neondb` |
| `DB_USER` | Database username | Database user |
| `DB_PASSWORD` | Database password | Database password |
| `PORT` | FastAPI backend port | `8000` |
| `VITE_API_BASE_URL` | Frontend API base URL | `http://localhost:8000/api` |