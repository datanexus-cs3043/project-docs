# Implementation & Integration Status - MedSync / CATMS

## Checkpoint & Evidence Boundary

**Reviewed: 2026-10-08, Asia/Colombo (UTC+05:30).** Supersedes the October 7 active status; earlier reviews retain their dated evidence.

| Repository | Local branch | Source checkpoint |
| :--- | :--- | :--- |
| CATMS-Backend | `main` | `140570083766a15ab970c65513183ba10beb5afd` |
| CATMS-Frontend | `main` | `018d7f3655c2b9d398cefa4073747e55a61d5514` |

Both application working trees were clean at inspection. No fetch/pull, application change or database access was performed for this documentation pass. Cached upstream agreement does not establish live remote state. New PRs/changes are user-reported; the available read-only PR search did not establish current states. This is not a live GitHub board, Neon or hosting audit.

**Status terms**:

- **Present in source**: A definition, route or interface was inspected. Runtime correctness is not implied.
- **Incomplete / mismatch**: Current source does not implement or align with the expected workflow.
- **Decision pending**: The business rule or permission needs agreement.
- **Unverified**: Relevant database/runtime/deployment evidence was not obtained.

---

## Current Implementation Summary

| Area | Current source evidence | Boundary / Remaining work |
| :--- | :--- | :--- |
| Stack & hosting | React/TypeScript/Vite, FastAPI, PostgreSQL/Neon; Docker defines backend/frontend only. | Hosted browser/API configuration and runtime operation remain unverified. |
| Roles & authorization | Five roles: admin, branch manager, doctor, combined receptionist/cashier and patient. Role/branch/ownership guards exist; public doctor-directory reads are retained. | Final action-level permissions and centralized-patient/cross-branch policy need agreement. See [Roles, Patient Self-Service & Report Output](roles_and_reports.md). |
| Authentication | Cookie-confirmed login, `/auth/me`, stale-authority checks and user-bound CSRF for selected doctor writes are present. | No backend public-registration endpoint. Generic frontend writes still omit CSRF. Account-link modelling remains a review topic; current staff lookup relies on login-audit associations. |
| Frontend data source | Browser-local database/mock fallback and demo login/role switching were removed. Date helpers are separate. | Old simulator and login type-error findings are resolved. Real contract failures remain; an empty page after a caught error is not proof of no records. |
| Doctors & specialties | List/detail/create/update integration and specialty creation/assignment corrections are present. Normalized specialty-name uniqueness has an index and separate migration. | Later directory/display-name/auth changes need current runtime verification. Neon index application is unknown. Public contact-field policy must not be inferred from nullable model fields. |
| Patient self-service | Patient pages, account links and several own-record API checks exist. | Account provisioning, permitted profile fields, nested URLs, generic CSRF, booking/lifecycle rights and clear errors need a selected future phase, not a new role. |
| Appointments | Protected CRUD, emergency/reschedule actions, notes and legacy single-treatment API contracts. SQL artifacts now add status, treatment links, lifecycle procedures and a booking exclusion constraint. | Complete/cancel now call `sp_complete_appointment` / `sp_cancel_appointment` (backend PR #13, `e66f5ed`) and responses include `status`; not yet runtime-verified against Neon. Presence of SQL definitions does not establish deployment or API integration. |
| Billing & compensation | Invoice/item and doctor-compensation APIs; SQL artifacts now define a separate `patient_payment` ledger and financial routines/triggers. | Existing `/payments` APIs still mean doctor compensation, not patient receipts. Automatic generation and reconciliation across all write paths are not established. |
| Insurance | Provider/policy/coverage/claim APIs and relationship validation; additional SQL validation definitions exist. | Frontend URLs/coverage contracts differ. Claim approval is not insurer cash settlement. |
| Reports & PDFs | Five management-only JSON handlers query base tables; six SQL views exist separately. | UI paths/fields, date/status semantics and financial definitions need alignment. PDF viewing/download is a deferred requirement; no inspected implementation provides it. |
| Database artifacts | Base file has 20 tables; later files add two tables/status, views, functions, procedures, triggers and additional constraints. | Repository SQL is not a cloud deployment record. Safe ordering, actual Neon objects and current full-stack behavior remain unverified. |

---

## Frontend / Backend Contract Gaps

All paths below are relative to `/api`. These comparisons are a review backlog, not approval to implement every correction.

| Workflow | Backend contract | Current frontend mismatch / boundary |
| :--- | :--- | :--- |
| Protected writes | Protected mutations require the appropriate CSRF dependency/header. | Selected doctor create/update helpers attach a user-bound token; generic post/put/delete helpers do not. |
| Complete / cancel | POST actions with PUT compatibility aliases; handlers lock the row, check scope and call the lifecycle procedures (PR #13). | UI PUT calls now match the alias. Requires the SQL 07 procedures to exist in Neon; end-to-end status transition not yet verified. Neither a method-only fix nor the SQL procedures alone completes this workflow. |
| Appointment creation | Required fields include patient, doctor, branch, local date/time, type and `created_by`; server derives actual creator attribution. Staff-only POST. | Forms may omit required fields; patient booking UI does not establish permission to create. |
| Appointment treatments | API still uses one `treatment_id`; removal uses that treatment ID. | UI expects bridge IDs/quantities. SQL now defines `appointment_treatment`, but API/UI integration remains unfinished. |
| Invoice generation | POST requires `appointment_id`, `staff_id`, `invoice_date` and `status`. | `generate()` sends only `appointment_id` and assumes automatic charge generation. |
| Payments | Invoice payment routes and `/payments/{id}` operate on `doctor_payment`. | UI models patient receipts and calls unsupported payment collections. The separate SQL receipt ledger does not change the existing API meaning. |
| Insurance | `/insurance/providers`, `/insurance/policies`, `/insurance/coverage`, `/insurance/claims`. | UI calls hyphenated top-level routes and nested policy coverages not provided by those handlers. |
| Patient insurance | `GET /patients/{id}/insurance` returns policy information with coverage. | UI calls `/patients/{id}/insurance-policies` and expects its own shape. |
| Contact deletion | `DELETE /patients/{patient_id}/emergency-contacts/{contact_id}`. | UI calls `/emergency-contacts/{contact_id}` without the parent. |
| Lookup / filter behavior | Endpoint-specific permissions, declared parameters and response schemas. | Some forms call restricted branch lists; several filters/joined fields are unsupported. Agree limited lookups rather than opening administration routes. |

No fallback should disguise these failures now. Verify the real response, authorization, method, schema and persistence when a workflow is selected.

### Report Contracts

All five routes are GET endpoints under `/api/reports`, restricted to admin/branch manager, with manager branch scoping. Doctor-own and combined front-desk/billing reports in the supplied reference are proposals, not current permissions.

| Backend route | Current result | Frontend difference |
| :--- | :--- | :--- |
| `/appointments-summary` | `total_appointments` and counts by appointment type. | Calls `/branch-appointment-summary`; expects dated branch/status rows. |
| `/doctor-revenue` | Per-doctor counts, billed and collected revenue from invoice amount/balance fields. | Same path, but other field names and undeclared date filters. |
| `/outstanding-balances` | Outstanding invoice rows with patient/date/balance/status. | Calls `/outstanding-patients`; expects patient aggregation. |
| `/treatments-by-category` | Category catalogue count, billed item quantity and item revenue. | Calls `/treatment-counts`; expects per-treatment fields/date filters. |
| `/insurance-vs-out-of-pocket` | Declared invoice payments/balances and claim approvals. | Calls `/insurance-summary`; different totals/fields. Approval and compensation are not cash receipts. |

The handlers do not call the new views or declare period filters. PDF output must use an agreed, authorized report dataset, not merely render whichever totals happen to exist. Definitions and future PDF acceptance are recorded in [Roles, Patient Self-Service & Report Output](roles_and_reports.md).

---

## Checks and Provenance

| Check | Result on October 8 | What it establishes |
| :--- | :--- | :--- |
| Backend Python AST parse | Passed for all 53 application files. | Syntax only; no application imports, startup, SQL execution or database connection. |
| Frontend TypeScript `--noEmit --incremental false` | Passed with existing dependencies. | Static type compatibility, not working API contracts or browser behavior. |
| Source/Git comparison | Reviewed at the checkpoints above. | Local definitions/history, not cloud equivalence or live PR approval. |

**Historical runtime evidence:** Earlier scoped checks included 121 FastAPI HTTP checks using disposable PostgreSQL 16.15, specialty mutation/race and migration checks, and a separate frontend harness/build. Those checks preceded later doctor/auth/layout edits and do not establish runtime correctness of the current checkpoints. They used isolated synthetic data, not Neon.

**Not performed in this pass:** dependency installation, production build, ESLint, Docker/service startup, browser/end-to-end execution, SQL execution, database queries/mutations or hosting verification. No new test files or application changes were made.

---

## Gradual Review Sequence — Not Implementation Authorization

Implementation is deferred pending the existing owners' handoff and an agreed next phase. This document records requirements without assigning members or taking over unfinished work.

1. **Refresh the handoff**: Check current PRs/diffs, owners, changed contracts and verification evidence. Preserve working contributions.
2. **Agree role and identity rules**: Resolve manager consultation-note actions, patient ownership/editable fields, branch context and report access. Obtain sanitized schema/deployment evidence before proposing database changes.
3. **Select one workflow**: Patient own-profile read is a candidate; appointment lifecycle/treatment integration is a separate candidate. Do not start all gaps together.
4. **Reconcile financial behavior**: Separate receipts, settlements, claim approvals and doctor compensation; compare API writes with the existing SQL definitions before adding objects.
5. **Validate one report, then PDF output**: Agree filters/grain/totals and server scope, align screen/JSON, then verify inline viewing and downloading. No PDF library or route is chosen yet.
6. **Verify delivery**: Review hosted API/cookie/CORS configuration and selected end-to-end workflows against the actual deployed schema.

For each selected phase, record its rule, affected files, checks, limitations and implementation/review evidence. Documentation updates alone do not approve application changes, migrations or merging.

---

## Source References

These links pin the reviewed source:

- [Backend routes and guards](https://github.com/datanexus-cs3043/CATMS-Backend/tree/140570083766a15ab970c65513183ba10beb5afd/app/api/v1/endpoints)
- [Backend request/response schemas](https://github.com/datanexus-cs3043/CATMS-Backend/tree/140570083766a15ab970c65513183ba10beb5afd/app/schemas)
- [Backend database artifacts](https://github.com/datanexus-cs3043/CATMS-Backend/tree/140570083766a15ab970c65513183ba10beb5afd/database)
- [Backend Compose topology](https://github.com/datanexus-cs3043/CATMS-Backend/blob/140570083766a15ab970c65513183ba10beb5afd/compose.yaml)
- [Frontend API services](https://github.com/datanexus-cs3043/CATMS-Frontend/blob/018d7f3655c2b9d398cefa4073747e55a61d5514/src/services/api.ts)
- [Frontend authentication](https://github.com/datanexus-cs3043/CATMS-Frontend/tree/018d7f3655c2b9d398cefa4073747e55a61d5514/src/auth)
