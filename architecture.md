# System Architecture - MedSync / CATMS

## Architectural Overview

MedSync / CATMS follows a multi-tier client-server architecture composed of a React frontend client, a Python FastAPI REST API backend, and a PostgreSQL relational database engine hosted on Neon (with local container support for offline development).

```mermaid
graph TD
    Client[Web Browser Client] -->|HTTP / REST API| Frontend[CATMS-Frontend: React / Vite / Nginx / Port 5173]
    Frontend -->|API Requests| Backend[CATMS-Backend: Python FastAPI / Port 8000]
    Backend -->|psycopg3 AsyncConnectionPool| Database[(PostgreSQL Database - Neon Cloud / Port 5432)]
```

---

## Component Breakdown

### 1. Presentation Layer (`CATMS-Frontend`)

- **Framework & Language**: React 19 with Vite and TypeScript.
- **Styling**: Tailwind CSS with responsive layout components and CSS styling.
- **State & Routing**: React Router v7, centralized `AuthContext` for session lifecycle, and `ProtectedRoute` enforcing role-based access control (`admin`, `branch_manager`, `doctor`, `receptionist_cashier`, `patient`).
- **HTTP Client**: Axios with `withCredentials: true` transmitting HttpOnly authentication cookies to FastAPI.
- **Production Build**: Multi-stage Docker build using `node:20-alpine` for asset compilation and `nginx:alpine` for static hosting.
- **Port Mapping**: Container port 80 mapped to host port 5173.

#### Key UI Modules:
- **App Shell**: Shared layout with responsive `Sidebar` and `Navbar` navigation.
- **Authentication**: `Login` component handling credential submission and session initialization.
- **Patient Management**: Central directory, registration form, and profile view.
- **Doctor Channeling & Rosters**: Practitioner listings, specialty filters, and appointment scheduling forms.
- **Billing & Finance**: Invoicing dashboard, itemized charges, and payment receipt recording.
- **Branch & Staff Oversight**: Multi-branch roster administration and operational analytics reports.

---

### 2. Application & API Layer (`CATMS-Backend`)

- **Runtime**: Python 3.11+.
- **Framework**: FastAPI (`uvicorn` ASGI server).
- **Data Access Layer**: Direct raw asynchronous SQL execution with `psycopg3` (`psycopg[binary,pool]`).
  - Implements `AsyncConnectionPool` with `row_factory=dict_row` and native `autocommit=True` connection pooling.
  - Retains explicit control over SQL queries, stored routines, transactions, and concurrency required for the CS3043 Database Systems module.
- **Authentication & Security**:
  - `argon2-cffi`: Password hashing via Argon2id algorithm.
  - `python-jose`: JWT token encoding/decoding.
  - HttpOnly secure cookies with CSRF token verification (`X-CSRF-Token` headers).
  - Role-Based Access Control (`Doctor`, `Staff`, `Manager`, `Patient`).
- **Dependencies**:
  - `fastapi`: High-performance async API framework.
  - `uvicorn`: ASGI web server implementation.
  - `pydantic` & `pydantic-settings`: Request validation, settings, and serialization.
  - `psycopg[binary,pool]`: PostgreSQL database driver and connection pool.
  - `python-dotenv`: Environment variable management.
- **Port Mapping**: Container/service port 8000 mapped to host port 8000.

#### Domain API Modules & Endpoints:
- **Authentication** (`/api/auth`): Login, logout, current user profile, CSRF token issuance.
- **Patients** (`/api/patients`): Patient registration, central directory lookups, emergency contact management.
- **Doctors & Specialties** (`/api/doctors`): Medical practitioner profiles, SLMC licensing, specialty mapping.
- **Appointments** (`/api/appointments`): Slot reservations, rescheduling, cancellations, clinical consultation notes.
- **Treatments** (`/api/treatments`): Medical service catalogue, category classifications, standard pricing.
- **Branches & Staff** (`/api/branches`, `/api/staff`): Multi-facility management, employee records, role assignments.
- **Billing & Payments** (`/api/billing`): Invoice generation, itemized charges, cash/card payment recording.
- **Insurance** (`/api/insurance`): Insurance providers, patient policy coverage, claim adjudication.
- **Operational Reports** (`/api/reports`): Management analytics querying PostgreSQL database views.

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
- **Container Network**: Docker Compose provides the default project network. Services communicate using Compose service names such as `postgres` and `backend`.
- **Health Checks**: Optional local PostgreSQL container uses `pg_isready -U ${DB_USER:-postgres} -d ${DB_NAME:-catms_db}` health check to ensure database readiness before backend startup.
- **Persistence**: Named Docker volume `postgres_data` attached to `/var/lib/postgresql/data` to ensure persistent storage across local container restarts. Cloud environments connect directly to Neon over TLS.

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