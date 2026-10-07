# Development Workflow & Environment Setup - MedSync / CATMS

## Prerequisites

- **Python**: Python 3.11 matches the backend Docker image; use virtual environments for local dependencies.
- **Node.js & npm**: A version compatible with the checked-out Vite package. The current Docker builder uses Node 24; the installed Vite package declares Node `^20.19.0 || >=22.12.0`.
- **Database Access**: An authorized PostgreSQL/Neon environment. Do not use the shared database for destructive setup or experimental writes.
- **Containerization**: Docker with Compose v2 for the containerized execution option.
- **Version Control**: Git.

Use the repository lockfile and requirements rather than installing unrelated test/build dependencies. Setup commands below are instructions for an approved development environment, not evidence that the current application has passed runtime verification.

---

## Local Development Execution

### Option 1: Application Services with Docker Compose

1. Keep the repositories side by side under a common workspace directory:

   ```text
   workspace/
     CATMS-Backend/
     CATMS-Frontend/
     project-docs/
   ```

   For a fresh workspace:

   ```bash
   git clone https://github.com/datanexus-cs3043/CATMS-Backend.git
   git clone https://github.com/datanexus-cs3043/CATMS-Frontend.git
   git clone https://github.com/datanexus-cs3043/project-docs.git
   ```

2. From `CATMS-Backend`, prepare backend configuration without overwriting an existing `.env`. PowerShell example:

   ```powershell
   if (-not (Test-Path .env)) { Copy-Item .env.example .env }
   ```

   Configure the authorized database connection, backend signing secrets, cookie settings, and allowed frontend origins. Never commit or paste actual credentials into documentation. Do not execute the numbered SQL files as an automatic setup step.

3. Start the two application services:

   ```bash
   docker compose up --build
   ```

   Stop them with:

   ```bash
   docker compose down
   ```

   The Compose file starts **backend and frontend only**. It does not create/reset Neon, run SQL migrations, or start a local `postgres` service. There is no project database-volume reset command.

4. Current endpoints:

   - **Frontend/Nginx**: `http://localhost:5173`
   - **FastAPI API**: `http://localhost:8000/api` by default
   - **Swagger Docs**: `http://localhost:8000/docs`
   - **OpenAPI JSON**: `http://localhost:8000/api/openapi.json`
   - **Database**: The external endpoint configured for the backend, not an automatically available `localhost:5432` service.

The frontend Docker build currently falls back to `http://localhost:8000/api`: frontend environment files are excluded, and no API-URL build argument is configured. This is unsuitable as a hosted-site address. See [System Architecture](architecture.md) before preparing deployment. Do not run Vite and Compose's frontend on the same host port simultaneously.

---

### Option 2: Component-Level Local Setup

Use separate terminals for the backend and frontend.

For a fresh backend checkout, create a virtual environment; otherwise use the existing one. PowerShell example, from `CATMS-Backend`:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Prepare backend configuration as above before starting Uvicorn. Startup opens the configured database connection; choose the authorized target first.

From `CATMS-Frontend`:

```bash
npm ci
npm run dev
```

Set a non-secret `VITE_API_BASE_URL` for local Vite when needed. The default is `http://localhost:8000/api`. Restart the dev server after changing configuration.

Service-specific references:

- **Backend**: [CATMS-Backend README](https://github.com/datanexus-cs3043/CATMS-Backend)
- **Frontend**: [CATMS-Frontend README](https://github.com/datanexus-cs3043/CATMS-Frontend)
- **Database**: [Database Design & Guidelines](database_design.md)

---

## Testing & API Verification

### Static and Build Checks

From `CATMS-Frontend`, after dependencies are already installed:

```powershell
.\node_modules\.bin\tsc.cmd --noEmit --incremental false
npm run lint
npm run build
```

- **TypeScript**: Run separately; `npm run build` currently invokes Vite only.
- **Lint**: Current ESLint configuration targets JS/JSX, not the TS/TSX application.
- **Build**: Produces assets; success does not prove working authentication, API contracts, or database transactions.
- **Backend**: Syntax/import/route checks are preliminary checks. Database-dependent operations need separately approved runtime verification.
- No project-wide automated test suite is established by these instructions. Do not install pytest/httpx or assume `pytest` validates the project merely because a previous guide listed them.

Current results and known failures are recorded in [Implementation & Integration Status](implementation_status.md).

### Interactive API Verification

On an explicitly approved development target:

1. Check the database-dependent `/api/health` response. Root metadata reporting `online` is not a database-health check.
2. Use `/docs` or `/api/openapi.json` to inspect actual methods, request fields, and response schemas.
3. Use authorized test accounts and verify role/ownership denial as well as allowed behavior. Do not include tokens, passwords, or patient data in logs or review messages.
4. For routes protected by CSRF, obtain a token from `GET /api/auth/csrf` and include `X-CSRF-Token` with the authenticated request. The frontend does not currently perform this write-header step.
5. Confirm real API and database results without browser-local fallback. A simulator response must not be recorded as a successful Neon transaction.

Completion/cancellation, patient receipts, automatic billing reconciliation, and some frontend contracts remain unresolved. Do not use demo success as their acceptance evidence.

---

## Team Collaboration & Git Standards

Use [GitHub Collaboration & Git Conventions](github_guidelines.md) for the draft collaboration guidance. Select one agreed correction, record its expected behavior, make a focused change, verify at the appropriate level, and update the affected document. A plan or documentation update does not itself approve an application or database change.
