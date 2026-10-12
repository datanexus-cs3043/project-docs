# Roles, Patient Portal & Reports — MedSync / CATMS

Source baseline: **12 October 2026**, backend `development@9e78cef`, frontend `development@2a941f6`. This document describes current source contracts, not unrestricted role grants or verified deployed permissions.

## Five roles and action scopes

`admin`, `branch_manager`, `doctor`, `receptionist_cashier` and `patient` are the five login roles. Reception and cashier duties are combined. Server checks remain authoritative even when a control is hidden in the UI.

| Operation | Current API scope |
| --- | --- |
| Register a patient; optional new portal account | Admin, branch manager, receptionist/cashier; assigned-branch scope for non-admin staff. |
| Patient directory/detail | Authorized front-desk/manager reads are cross-branch; registration branch is a filter, not a read boundary. Doctors need a related own-branch appointment; patients access their linked record, not the directory. |
| Book, cancel, reschedule, emergency walk-in, complete | Staff roles only; branch checks and doctor-own appointment checks apply. Patients cannot perform scheduling mutations. |
| Read own appointment/treatments | Linked patients; authorized scoped staff. |
| Record/remove visit treatments | Staff roles, scoped to the appointment; doctors only their own visits. Cancelled or invoiced visits are locked. |
| Consultation note read/write | Admin, branch manager and doctor under appointment scope; not receptionist/cashier or patient. Cancelled/invoiced visit edit guards apply. |
| Generate invoice; record patient receipt | Admin, branch manager, receptionist/cashier; invoice-branch scope. |
| Read invoices | Authorized scoped staff, doctor-own visits and patient-own invoices. Doctor invoice visibility does not grant receipt-ledger administration. |
| Patient receipt history | Billing roles; patients may read receipt history for their own invoice. Doctors do not have the patient-receipt API role. |
| Doctor compensation | Staff read under branch/doctor-own checks; writes restricted to admin, manager and receptionist/cashier. Not a patient receipt. |
| Policy, coverage and claim writes | Admin, manager, receptionist/cashier with record/branch checks. Patients read own insurance; doctor claim reads are limited to own visits. |
| Treatment/category, specialty and provider catalogue writes | Admin; catalogue visibility does not imply editing rights. |
| Branch management | Admin creates/deletes and assigns managers; manager may update permitted own-branch fields. |
| Staff management | Admin/manager; managers cannot manage privileged accounts or assign admin/manager roles/account links. |
| Global account administration | Admin. Self-service account actions use dedicated protected routes, not global account rights. |
| Management reports | Admin and branch manager only; managers cannot select another branch. |

This table summarizes the important workflows, not every endpoint's CRUD behavior. Patient self-updates are restricted to own contact details, email and address; staff demographic/contact writes retain registration-branch scope, and doctor directory access does not grant demographic editing. Emergency contacts and insurance information share the authorized cross-branch read contract. Manager consultation-note authoring is an existing API permission, not merely read-only; confirm that clinical policy during acceptance rather than describing a desired restriction as enforced.

## Patient portal and scheduling

Public self-registration is not implemented. The sign-in page provides account help rather than sending an unsupported registration request.

During **new staff-managed registration**, a portal username plus valid account email can create a linked account together with the patient and optional emergency contact. Omitting the portal option creates only the patient record. Existing records/accounts are not rewritten. Initial QA access follows the project's default-password policy; the interface instructs password change, but first-login rotation is not enforced.

Login accepts a trimmed, case-insensitive username or **account email** using the existing JSON `username` field. Demographic/staff contact email and account email are different records: editing a contact email does not necessarily change sign-in identity. New account writers check both identifiers against both account columns. Ambiguous legacy matches fail closed rather than selecting an arbitrary account.

Patients use their linked profile, appointment history, invoices and insurance information. Scheduling remains staff-managed; patient self-booking, cancellation and rescheduling are intentionally not enabled.

A central patient database permits shared patient/emergency/insurance reads for authorized front-desk/manager roles. It does not mean unrestricted cross-branch writes or clinical/financial access. Doctors' patient-record access follows their appointment relationship; appointment, invoice and claim operations retain their own branch/doctor/ownership rules.

## Appointment-to-billing rules

- Appointment states are Scheduled, Completed and Cancelled. Cancellation/completion act on Scheduled visits; rescheduling preserves history rather than overwriting it as an unrelated booking.
- Record consultation/treatment evidence before completion; further recording on a Completed unbilled visit is supported. Invoice generation requires at least one captured treatment on a Completed visit. A note-only visit needs its actual consultation/service charge recorded before invoicing.
- Multiple treatments carry quantities and unit prices copied at recording time; later catalogue price changes do not silently alter the visit.
- Generate one invoice per appointment. Issued items cannot be modified through item POST/PUT/DELETE routes.
- Record receipts through `POST /api/invoices/{invoice_id}/patient-payments`, not legacy doctor-payment URLs. Derived totals/status cannot be edited directly.
- If a receipt was acknowledged but refresh failed, refresh/reconcile without paying again. If its network/server outcome is unknown, inspect the ledger before retrying; the system does not claim automatic exactly-once payment delivery.

The database ledger supports refund/adjustment records, but there is no dedicated public correction workflow claimed here. Do not correct an issued invoice by bypassing API protections.

## Insurance interpretation

MedSync uses an explicit **per-policy/invoice (visit)** cap for each covered treatment. This is the project's bounded data model, not a claim about every insurer's contractual rules or regulatory compliance.

1. The policy belongs to the invoice's patient and must be active/valid for the invoice date for eligible non-rejected claims.
2. Group repeated invoice lines by treatment before applying coverage percentage and `maximum_amount`; use issued prices, not today's catalogue price.
3. Floor each treatment entitlement to cents. Missing, zero or invalid coverage does not grant a benefit; duplicate coverage rules fail closed.
4. Pending claims reserve their requested amounts; Approved claims reserve approved amounts; Rejected claims reserve zero.
5. Aggregate reservations must fit the policy's eligible amount for this invoice and the invoice-wide billed ceiling across policies.
6. Partial approval preserves the original requested amount. A filed claim cannot move to another invoice/policy.
7. Claim approval does not create insurer cash settlement, patient receipts or a balance reduction.

Safe rejection of legacy excessive/inactive-policy claims requires the new SQL trigger behavior. The matching function/trigger was validated and installed on the shared Neon target on 12 October 2026; every existing claim passed status-update revalidation. Competing-session and browser acceptance remain separate checks. See [Database Design](database_design.md).

Annual benefit pools, waiting/preauthorization rules, automated insurer communication and historical coverage versioning are not modeled. Later policy-rule edits can affect subsequent validation/estimates; do not present estimates as frozen historical insurer decisions.

## Five management reports

Routes below are GET endpoints under `/api/reports`. All support optional `from_date`, `to_date` and admin-selectable `branch_id`; managers remain on their assigned branch. Invalid reversed date ranges are rejected. Date bounds are inclusive.

| Route | Grain, period and principal fields |
| --- | --- |
| `/appointments-summary` | One branch/day; appointment date. `scheduled`, `completed`, `cancelled`, `total`. Optional `date` selects one day and overrides from/to. |
| `/doctor-revenue` | Doctor/branch; invoice date. `appointment_count`, `total_revenue` (billed), `collected` (patient paid), `avg_revenue`. This is not doctor compensation or all scheduled visits. |
| `/outstanding-balances` | Patient aggregate over matching outstanding invoices; invoice date. `total_outstanding`, `invoice_count`, `oldest_invoice_date`. The oldest date is within the selected invoices, not necessarily lifetime debt. |
| `/treatments-by-category` | Treatment within category on Completed visits; appointment date. `count` is summed treatment quantity, `total_revenue` is captured treatment value, not cash collected or catalogue size. |
| `/insurance-vs-out-of-pocket` | Totals over scoped invoices by invoice date. `invoiced_amount`, `insurance_approved_amount`, `patient_paid_amount`, `outstanding_amount`, compatibility `out_of_pocket_amount`. |

`patient_paid_amount` is net patient ledger receipts. `out_of_pocket_amount` remains an alias of that amount for compatibility; it is **not** billed minus approved insurance. The UI's **estimated patient exposure** is billed minus approved claims and is labeled as an estimate, not a paid amount or settled balance. Report period filtering selects invoices/visits; it does not automatically turn current totals into an as-of-date financial history.

Doctor-own and front-desk management reports are not enabled. Their record/statement access does not grant the global report routes.

## Print / Save as PDF

The Reports page uses the loaded dataset and its applied filter snapshot, escapes dynamic export text and opens a native browser print window. Users select a printer or **Save as PDF**. Changing an unapplied filter does not relabel previously loaded data.

There is no backend PDF endpoint, attachment response or automatic PDF file download. Keep this distinction in demos and user instructions. PDF creation depends on browser capability and popup/print permissions.

Acceptance must check all five reports against known scenarios, screen/export totals, applied scope/period, empty/error states, long-table page breaks and readable headers/units. Browser print/PDF layout has not been certified at this source checkpoint.
