# Development Workflow & Environment Setup — MedSync / CATMS

## Working baseline

Use the exact **`development`** branch in both application repositories. They have independent histories and should be kept side by side:

```text
workspace/
  CATMS-Backend/
  CATMS-Frontend/
  project-docs/
```

Before editing or pulling, check each repository's branch, status, current diff and intended base. Preserve contributors' uncommitted work. Where the working tree and local history permit it, update the agreed branch with `git pull --ff-only origin development`; stop on divergence rather than reset or discard changes.

Hosted main is a separate release baseline. The [GitHub Guidelines](github_guidelines.md) describe the main-merge authorization boundary.

## Prerequisites and configuration

- Python 3.11 and a backend virtual environment.
- Node/npm compatible with the checked-out Vite version; the frontend Docker builder uses Node 24.
- Docker with Compose v2 if using containers.
- Git and authorized PostgreSQL access. Compose does not provide a database.

Use the existing lockfile/requirements. Do not install unrelated tooling merely to reproduce an old guide.

From a fresh backend checkout, prepare configuration without overwriting an existing file:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

Configure the authorized database endpoint/TLS, distinct signing secrets, cookie policy and exact frontend CORS origins. Keep credentials untracked. See [Architecture](architecture.md#configuration-contract) for names and responsibilities.

For frontend Vite, a non-secret API setting may be placed in a local untracked environment file:

```dotenv
VITE_API_BASE_URL=http://localhost:8000/api
```

Vite reads it at build/startup time. Restart the dev server after changes; rebuild for a different hosted API. Never include database or backend secrets in frontend variables.

**Do not run the numbered SQL sequence as automatic setup.** Obtain the authorized target's schema/migration state first. The current shared dataset must be preserved; see [Database Design](database_design.md#safe-database-change-process).

## Option A: Docker Compose

From `CATMS-Backend`:

```bash
docker compose up --build
```

Stop the application services with:

```bash
docker compose down
```

The current services are backend and frontend only. No SQL migration, seed, database reset or PostgreSQL volume is included. Source is copied into the images; rebuild changed services to run new code rather than assuming hot reload.

The October 12 public directory change requires both development applications:
the frontend calls `/api/public/clinic-directory`, which older hosted backend builds
do not contain. Database changes alone do not install that endpoint. Rebuild/run
the development backend and frontend together, then check directory loading and
admin catalogue/branch updates; no hosting or main-release step is implied.

| Component | Default local address |
| --- | --- |
| Frontend/Nginx | `http://localhost:5173` |
| API | `http://localhost:8000/api` |
| Swagger | `http://localhost:8000/docs` |
| OpenAPI JSON | `http://localhost:8000/api/openapi.json` |
| PostgreSQL | External configured endpoint; not an automatically available localhost service. |

Do not run the Vite server and Compose frontend on the same port. The Docker frontend excludes `.env` and `.env.local` and has no API-address build argument; its localhost fallback suits the local browser arrangement, not cloud hosting. Review other environment filenames before packaging. Nginx is a SPA static server, not an API reverse proxy.

## Option B: Local application terminals

From `CATMS-Backend`, create a virtual environment only if one is not already available:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Configure the target before starting: application startup opens the database pool.

From `CATMS-Frontend` in another terminal:

```bash
npm ci
npm run dev
```

Use `npm ci` for a fresh/dependency-changed checkout; an already installed environment does not require reinstalling on every edit.

## Verification levels

From the frontend with dependencies installed:

```powershell
.\node_modules\.bin\tsc.cmd --noEmit --incremental false
npm run lint
npm run build
```

- TypeScript checks are separate: the build script runs Vite only.
- The current ESLint file pattern covers JS/JSX, not the application's TS/TSX; a lint pass is not comprehensive TypeScript lint coverage.
- A build emits assets but does not exercise sessions, permissions or database transactions.
- Backend syntax/import/OpenAPI checks are preliminary; avoid loading real credentials or connecting a shared database just to check syntax.
- Existing SQL assertions are reviewed/run only on an approved target. No project-wide automated acceptance suite is implied by these commands.

Current recorded results and limitations belong in [Implementation Status](implementation_status.md).

### Full-stack QA

On an approved synthetic test environment:

1. Check `/api/health` for database connectivity and inspect actual methods/fields in OpenAPI.
2. Confirm login with username and account email, session restoration, logout and password change. Protected non-auth writes use the shared CSRF interceptor. Direct API tools must obtain `/api/auth/csrf` and send `X-CSRF-Token` with the authenticated request.
3. Run staff registration with/without portal access, including collision and failed-contact rollback.
4. Exercise normal/walk-in booking, overlap denial, cancellation/rescheduling and date/time round trips. Verify scheduling remains unavailable to patients.
5. Record care, complete the visit, generate one invoice and record a partial receipt. Confirm treatments/items lock after invoicing and totals persist after refresh.
6. Exercise covered/uncovered claims, pending reservations, partial approvals and rejection after the approved trigger migration. Approval must not silently become a receipt.
7. Reconcile all five report totals and filter scopes, then inspect native print/save-as-PDF output.
8. Repeat own/other patient, doctor and branch denial cases; check loading/errors, duplicate clicks, route refresh and stale responses.

For an unknown receipt outcome, inspect the existing ledger before retrying. Do not create a second payment merely to test recovery.

Record source SHAs, database migration/version, role, steps and expected/actual outcomes. Share sanitized evidence, not patient details, passwords, cookies or raw connection logs. The acceptance checklist in [Implementation Status](implementation_status.md#team-acceptance-handoff) is the common handoff record.

## Change handoff

Select a coherent workflow, review current teammate changes, agree the contract, implement and verify in proportion to risk, and update the affected shared document. Record application implementation, schema application, review and runtime acceptance separately. A documentation change does not grant permission to migrate shared Neon or release main.
