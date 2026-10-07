# Implementation & Integration Status - MedSync / CATMS

## Checkpoint & Evidence Boundary

**Reviewed: 2026-10-07, Asia/Colombo (UTC+05:30).**

| Repository | Local branch | Source checkpoint |
| :--- | :--- | :--- |
| CATMS-Backend | `main` | `37fcdcd2be4556147203b574fd92b5a6aa93a261` |
| CATMS-Frontend | `main` | `079780d8c12f20269f080ac34df33273030481b7` |

This is a local source review, not a live GitHub board, Neon, or hosting audit. Both application working trees were clean at inspection. The backend includes the merged API work; its presence on `main` does not establish full delivery.

**Status terms**:

- **Present in source**: A definition, route, or interface was inspected. Runtime correctness is not implied.
- **Incomplete / mismatch**: Current source does not implement or align with the expected workflow.
- **Decision pending**: The business rule or permission needs explicit agreement.
- **Unverified**: The relevant database/runtime/deployment evidence was not obtained.

---

## Current Implementation Summary

| Area | Current source evidence | Boundary / Remaining work |
| :--- | :--- | :--- |
| Stack & hosting | React/TypeScript/Vite, FastAPI, PostgreSQL/Neon; Docker defines backend/frontend only. | Hosted browser/API configuration and runtime operation are unverified. |
| Roles & authorization | Five declared roles; backend role, branch, and ownership checks on clinical/domain routes. Public doctor-directory reads are retained. | Final permission matrix, reliable account linkage, session freshness, and centralized-patient access still need review. Client role checks are not security enforcement. |
| Authentication | Login/logout/me/CSRF endpoints and HttpOnly-cookie support. | Frontend login call has a type error; public registration has no backend endpoint. Protected frontend writes omit the CSRF header. |
| Frontend domain pages | Patient, doctor, appointment, invoice, insurance, staff, branch, and report interfaces plus service wrappers. | Not merely an initial visual prototype, but not fully integrated with the real API either. |
| Demo / fallback | Browser-local simulator supplies responses on network errors and non-auth 404/405/501. AuthContext can enter a demo session after a real login rejection. | Demo must be explicitly distinguished from real operation. It is not isolated from network-first requests or an existing real cookie; potential real requests must not be treated as harmless demo activity. |
| Appointments | Protected CRUD, emergency/reschedule actions, notes, and a single treatment assignment. | No persisted status; authorized complete/cancel requests reach 501. Normal create/update do not use the emergency/reschedule overlap guard. No verified multiple-treatment clinical relation. |
| Billing & compensation | Invoice/item CRUD and doctor compensation endpoints. | No patient-receipt ledger in the local schema; automatic invoice generation and full reconciliation are not established. |
| Insurance | Provider/policy/coverage/claim CRUD and relationship validation. | Frontend URLs and coverage expectations differ. Claim approval is not insurer cash settlement. Simulator allocation/payout formulas are not recorded business rules. |
| Reports | Five management report handlers query base tables. | Frontend paths, fields, and report grain differ. Date/status semantics and final report acceptance need agreement and verification. |
| Database artifacts | 20 table definitions, separate manager FK, and 26 index statements. Several routine/report/test files remain intentional placeholders. | Actual Neon objects, schema equivalence, deployed routines, and concurrency behavior are unverified. |

---

## Frontend / Backend Contract Gaps

All paths below are relative to `/api`. These comparisons describe current source, not instructions to implement every correction at once.

| Workflow | Backend contract | Current frontend mismatch |
| :--- | :--- | :--- |
| Login | AuthContext function accepts `username, password`; API login accepts a credentials object. | Login UI passes one object to AuthContext's two-argument function. |
| Protected writes | Routes using `require_csrf` require `X-CSRF-Token`. | Token-fetch helper exists but generic write helpers do not attach the header. |
| Complete / cancel | `POST /appointments/{id}/complete` and `POST /appointments/{id}/cancel`; currently 501 after authorization/resource checks. | UI sends PUT and expects a persisted status transition. Changing the method alone does not implement the lifecycle. |
| Appointment creation | Required fields include patient, doctor, branch, date/time, type, and `created_by`; server derives actual creator attribution. | Forms may omit required fields. The API schema still requires `created_by`, despite server-side attribution. |
| Appointment treatments | One `treatment_id`; listing returns treatment objects; removal uses `treatment_id`. | UI expects bridge IDs and per-appointment quantities. |
| Invoice generation | POST requires `appointment_id`, `staff_id`, `invoice_date`, and `status`. | `generate()` sends only `appointment_id` and assumes automatic charge generation. |
| Payments | Invoice payment routes and `/payments/{id}` represent doctor compensation, with doctor/appointment/date/time/amount fields. | UI models patient receipts with amount/method and calls unsupported payment collections. This is a semantic mismatch, not just a URL rename. |
| Insurance | `/insurance/providers`, `/insurance/policies`, `/insurance/coverage`, `/insurance/claims`. | UI calls hyphenated top-level routes and nested policy coverages not provided by those handlers. |
| Patient insurance | `GET /patients/{id}/insurance` returns policy information with coverage. | UI calls `/patients/{id}/insurance-policies` and expects its own response shape. |
| Contact deletion | `DELETE /patients/{patient_id}/emergency-contacts/{contact_id}`. | UI calls `/emergency-contacts/{contact_id}` without the patient parent. |
| Lookup / filter behavior | Endpoint-specific role permissions, declared parameters, and response schemas. | Some forms call the admin/manager-only branch list; several UI filters and joined fields are not supported by the corresponding backend list responses. |

A successful local fallback can mask these differences. Inspect the real response method, schema, origin, and persistence when validating a workflow.

### Report Contracts

All five report routes are GET endpoints under `/api/reports`, restricted to admin/branch manager with branch scoping for managers.

| Backend route | Current result | Frontend difference |
| :--- | :--- | :--- |
| `/appointments-summary` | Object with `total_appointments` and counts by appointment type. | Calls `/branch-appointment-summary`; expects dated branch/status rows. |
| `/doctor-revenue` | Per-doctor counts, `billed_revenue`, and `collected_revenue`, derived from declared invoice amount/balance fields. | Same path, but expects other field names and date filtering that the handler does not declare. |
| `/outstanding-balances` | Outstanding invoice rows with patient, date, balance, and status. | Calls `/outstanding-patients`; expects per-patient aggregation. |
| `/treatments-by-category` | Per-category catalogue count, billed item quantity, and item revenue. | Calls `/treatment-counts`; expects per-treatment fields and date filters. |
| `/insurance-vs-out-of-pocket` | Invoice amounts, claim approvals, declared patient payments, and outstanding balance. | Calls `/insurance-summary`; expects different totals/fields. Neither claim approval nor doctor compensation should be described as insurer/patient cash receipts. |

These are current result definitions, not a declaration that they satisfy every required report or are financially reconciled.

---

## Checks Performed

| Check | Result on 2026-10-07 | What it establishes |
| :--- | :--- | :--- |
| Backend Python AST parse | Passed for 53 application files. | Python syntax only; no imports, service startup, SQL execution, or database connection. |
| Frontend TypeScript `--noEmit --incremental false` | Failed: `src/auth/Login.tsx:60`, TS2554, expected two arguments but received one. | The current source does not pass the type check. |
| Existing ESLint | Exit 0. | Current configuration targets JS/JSX; this is not validation of the TS/TSX application. |
| Source/config/schema comparison | Reviewed against the commits above. | Repository definitions and contract differences, not live cloud equivalence. |

The latest frontend commit fixes the `Myappointments` import casing; that earlier finding is no longer unresolved at this checkpoint.

**Not performed**: dependency installation, production asset build, Docker startup, browser/end-to-end execution, database queries/mutations, SQL/concurrency checks, hosting verification, or live remote-state refresh. No new tests or application changes were made in this documentation pass.

---

## Gradual Correction Sequence

This is a proposal for team agreement, not approval to execute or an assignment of members.

1. **Separate real and demo operation**: Make data provenance explicit and prevent demo activity from sharing real-request authority. Then correct the focused login/session and CSRF integration.
2. **Agree access and identity rules**: Confirm stable account associations, role-specific permissions, and centralized-patient/cross-branch behavior. Obtain authorized, sanitized Neon schema evidence before proposing SQL changes.
3. **Complete one clinical workflow**: Decide appointment status, reschedule history, time/conflict rules, and treatment cardinality; align API and UI for that selected workflow.
4. **Reconcile billing and insurance**: Define patient receipts, settlement, doctor compensation, totals, and balances before adopting simulator formulas or adding persistence.
5. **Validate reports and delivery**: Align required report definitions and UI contracts, record verified SQL artifacts, and verify hosted API/cookie/CORS configuration and end-to-end behavior.

For each step, record the agreed rule, affected files, actual checks, known limitations, and contributor/reviewer evidence. Keep unrelated feature work out of the selected correction.

---

## Source References

These links pin the reviewed source, so later changes on `main` do not silently alter this checkpoint:

- [Backend routes and guards](https://github.com/datanexus-cs3043/CATMS-Backend/tree/37fcdcd2be4556147203b574fd92b5a6aa93a261/app/api/v1/endpoints)
- [Backend request/response schemas](https://github.com/datanexus-cs3043/CATMS-Backend/tree/37fcdcd2be4556147203b574fd92b5a6aa93a261/app/schemas)
- [Backend database artifacts](https://github.com/datanexus-cs3043/CATMS-Backend/tree/37fcdcd2be4556147203b574fd92b5a6aa93a261/database)
- [Backend Compose topology](https://github.com/datanexus-cs3043/CATMS-Backend/blob/37fcdcd2be4556147203b574fd92b5a6aa93a261/compose.yaml)
- [Frontend API services](https://github.com/datanexus-cs3043/CATMS-Frontend/blob/079780d8c12f20269f080ac34df33273030481b7/src/services/api.ts)
- [Frontend authentication](https://github.com/datanexus-cs3043/CATMS-Frontend/tree/079780d8c12f20269f080ac34df33273030481b7/src/auth)
- [Frontend demo implementation](https://github.com/datanexus-cs3043/CATMS-Frontend/blob/079780d8c12f20269f080ac34df33273030481b7/src/services/localDb.ts)
