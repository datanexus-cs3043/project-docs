# Roles, Patient Self-Service & Report Output - MedSync / CATMS

**Scope clarification: 2026-10-08, Asia/Colombo (UTC+05:30).** This document separates intended responsibilities, observed implementation and decisions still required. It is not a grant of API permissions or a claim of lecturer approval.

## Five Application Roles

| Role identifier | Responsibility |
| :--- | :--- |
| `admin` | Authorized system-wide administration, accounts, catalogues and oversight. |
| `branch_manager` | Authorized operations, staff oversight and reporting within the assigned branch. |
| `doctor` | Assigned appointments/patients, consultation records and clinical treatment recording. |
| `receptionist_cashier` | Combined front-desk registration/scheduling and authorized billing/insurance operations. |
| `patient` | Own authorized profile, appointments, bills and insurance information. |

Receptionist and cashier are **one login role**, not two. Patient is already declared in both applications; its incomplete functionality does not make it a newly added sixth role. Staff classifications and historical SRS actors such as nurse/insurance officer do not automatically create additional login roles.

The submitted SRS separates several staff responsibilities and leaves detailed permissions open. The October 8 staff feature reference also separates reception/cashier and omits patient. The matrix below consolidates that reference and adds patient scope; it is a **review proposal**, not implemented or universally approved access. A checkmark must not be interpreted as unrestricted create/read/update/delete.

## Normalized Responsibility Matrix — Proposed

Every permission is additionally subject to record ownership, branch/clinical scope, approved actions and history-retention rules. “Manage” is not permission to erase clinical or financial history.

| Feature | Admin | Branch manager | Doctor | Receptionist / cashier | Patient |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Dashboard | System oversight | Branch overview | Own practice | Front-desk/billing overview | Own care overview |
| Patients | Authorized administration | Branch operations under centralized-record policy | Assigned clinical relationship | Registration/search and permitted demographic updates | Own profile only |
| Appointments | Authorized administration | Branch coordination | Own roster and agreed clinical updates | Scheduling, coordination and staff walk-ins | Own history; booking/cancel/reschedule rights to agree |
| Consultation notes | Authorized clinical oversight | Reference proposes View; need-to-know/action policy to agree | Notes for own assigned appointments | No clinical-note access | No raw notes by default; released own documents need a separate decision |
| Treatments | Catalogue administration | Catalogue view | Select/record appointment treatments; not implied catalogue creation | Catalogue view | Service/pricing view; no clinical treatment authoring |
| Invoices | Authorized administration | Branch view | Authorized own-appointment view | Approved invoice/billing operations | Own invoices/balances |
| Payments | Authorized financial operations | Reference proposes View; actions to agree | Own compensation statement policy to agree | Approved patient-receipt work; payout rights considered separately | Own payment information; no payout administration |
| Insurance | Authorized administration | Branch view | Relevant clinical/coverage view | Approved policy/claim processing | Own policy/coverage/claim information; no approval authority |
| Staff | Authorized administration | Assigned-branch oversight with privileged-account restrictions | No staff administration | No staff administration | No access |
| Branches | Authorized administration | Own-branch management/view | Own operational context | Own operational context | Necessary public clinic information only |
| Reports | Authorized system reports | Branch-scoped management reports | Own-data reports proposed, not yet supported | Branch-operational/billing reports proposed, not yet supported | Own documents only; no global management reports |
| User management | Authorized account administration | No global account administration | No global account administration | No global account administration | Own permitted profile only; no role/privilege editing |

Combining reception/cashier duties does not combine all possible financial and clinical privileges. Catalogue treatment creation, appointment treatment recording, patient receipts and doctor compensation are different operations.

## Differences from Current Source

Checkpoint: backend `1405700`, frontend `018d7f3`; see [Implementation Status](implementation_status.md).

| Topic | Observed source | Review needed before a change |
| :--- | :--- | :--- |
| Reports | API `REPORT_ROLES` and `/reports` UI allow only admin/branch manager; managers are branch-scoped. | Doctor-own/front-desk/billing reports require explicit datasets and server scopes, not just another menu entry. |
| Manager consultation notes | Current note read/create/update/delete handlers allow admin, branch manager and doctor, with scope checks. | Reference proposes manager View only; agree clinical need-to-know, authoring and retention policy rather than assuming current writes are the desired rule. |
| Doctor payments | Payment API lets doctors read their own compensation; writes are restricted to admin/manager/receptionist_cashier. | Reference's “no payments” is ambiguous. Separate own compensation visibility from patient receipts and payout administration. |
| Patient booking | UI includes booking, but appointment POST is staff-only. Complete/cancel are implemented for staff roles (PR #13). | Agree patient actions, account ownership, lifecycle and concurrency before connecting controls. |
| Branch context | Branch administration/list UI is restricted; staff/patient flows sometimes need limited lookup information. | A branch context requirement does not justify opening all branch/staff management endpoints. |
| Patient profile | Own-record API checks and personal pages exist; generic writes omit CSRF and some nested URLs differ. | Confirm editable-field allowlists, ownership, correct methods/URLs and honest error handling. |
| Doctor catalogue | Public backend directory reads coexist with a UI that excludes doctor-role directory navigation. | Distinguish navigation choices from intended public field/access policy. |

Client routing/hiding buttons is not authorization. API queries and exports must enforce role, record and branch scope independently. Centralized patient records do not imply unrestricted clinical or financial access across branches.

## Deferred Patient Functionality

Patient implementation is **not being started in this documentation pass**. Preserve the existing pages and wait for an agreed owner handoff and a selected implementation phase.

1. Confirm account provisioning and a stable account-to-patient association. Public registration UI exists, but `/auth/register` is absent from the backend.
2. Validate own-profile read, then agree editable contact/address fields and emergency-contact operations. Do not permit role, account-link or branch reassignment through self-editing.
3. Validate own appointments and doctor/service browsing. Decide whether patients may book, cancel or reschedule, and connect only the approved actions to persisted lifecycle rules.
4. Validate own invoices, balances and insurance information. Patient receipts, insurer settlement and doctor compensation remain separate.
5. Check real persistence, CSRF on writes, clear loading/errors, missing links, and denial when another patient's ID is substituted. Do not claim success from an empty screen after a failed request.

Access to released clinical summaries or personal PDF documents is a separate permission/content decision. It does not grant patient access to staff management reports or unrestricted consultation notes.

## Required Reports and PDF Output — Deferred

The official project brief names five reports. **PDF generation for viewing and downloading is an additional delivery clarification recorded on October 8**, not an attributed official export requirement. Exact layouts/export formats were left open in the SRS.

| Report | Data/meaning that must be settled before export |
| :--- | :--- |
| Daily branch appointment summary | Branch/date/timezone and Scheduled/Completed/Cancelled counts. |
| Doctor revenue | Doctor/branch/period and explicit billed-versus-collected revenue; do not substitute doctor payouts. |
| Patients with outstanding balances | Patient/invoice grain, actual receipts, insurance treatment and non-duplicated totals. |
| Treatments by category over a period | Recorded treatment quantity, relevant date and category, not merely catalogue size. |
| Insurance versus out-of-pocket | Distinguish coverage, claimed/approved amounts, insurer settlement, actual patient payments and balances. |

### Future implementation acceptance

- Correct and authorize one report's data first; use the same definitions/filters for its on-screen results and PDF. Existing report URLs/fields do not all match the frontend.
- Provide both authenticated inline viewing and attachment download of a valid `application/pdf` response, with a useful filename. Route design/rendering technology are not chosen here.
- Include report title, applied period/filters, data scope, generated-at time with timezone, units/currency, totals and suitable page numbering. Long tables, empty results and page breaks must remain readable.
- Apply server-side role/ownership/branch restrictions before querying/rendering. Never widen access through the PDF route or trust a client-selected role, patient or branch as authority.
- Include only needed patient/financial details. Escape unsafe text for the chosen renderer; avoid unapproved persistent/public exports and shared-cache leakage of sensitive output.
- Verify totals against known scenarios, table/PDF agreement, denied access, file validity and visual layout. A valid PDF alone does not prove correct report figures.

Current source has five JSON report handlers and no inspected PDF generation/delivery implementation. SQL views exist, but those API handlers still query base tables. No PDF dependency, backend/frontend code or live database change is authorized by this document.

## Collaboration and Review Gate

Existing owners should finish and hand over their selected work before cross-cutting correction. Confirm task/contract ownership, review current PRs, preserve useful code and correct one issue at a time. Record implementation, review, integration and documentation separately; neither commit/line counts nor merging another person's work establish feature authorship.

This scope note does not assign members, approve a PR, authorize a migration or move the delivery deadline. See [Project Overview](project_overview.md), [Database Design](database_design.md) and [Implementation Status](implementation_status.md) for provenance and limits.
