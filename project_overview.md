# Project Overview — MedSync / CATMS

## Purpose and context

MedSync / CATMS is a database-centered outpatient clinic management system developed by the five-member DataNexus team for CS3043 Database Systems at the University of Moratuwa. It brings patient registration, branch operations, clinical visits, billing and management reporting into one relational workflow.

The submitted SRS and ER diagrams are design references. Later schema and workflow decisions must be reconciled with those references; implementation changes are not automatically evidence of design approval. The current application uses React/TypeScript, FastAPI and PostgreSQL hosted on Neon.

This overview describes the **development source baseline of 12 October 2026**. [Implementation Status](implementation_status.md) distinguishes that baseline from deployment and final acceptance.

## Core workflow

1. **Register a patient:** authorized staff create the demographic record and optional emergency contact. New registration can also provision a linked patient portal account with a unique username and account email.
2. **Schedule a visit:** staff select the patient, doctor, branch, appointment date/time and type. Normal bookings, emergency walk-ins, cancellation and rescheduling remain staff-managed. Rescheduling retains the relationship to the original visit.
3. **Record care:** authorized staff record treatment quantities and captured catalogue prices; authorized clinical users maintain consultation notes. Doctors are scoped to their own appointments.
4. **Complete and invoice:** completion requires recorded clinical evidence. Invoice generation additionally requires a completed visit with at least one recorded treatment. One invoice is generated for the visit, with itemized charges copied from the treatment records.
5. **Record patient receipts:** authorized billing staff record full or partial payments. Invoice paid amount, balance and status are synchronized with the patient ledger. Doctor compensation is a separate record.
6. **Process insurance:** staff manage patient policies, treatment coverage and manual claims. Pending, Approved and Rejected claims have distinct reservation rules; approval is not insurer settlement or a patient receipt.
7. **Review reports:** administrators and branch managers inspect scoped appointment, revenue, outstanding-balance, treatment and insurance summaries, with browser print/save-as-PDF output.

## Five application roles

| Identifier | Primary responsibility |
| --- | --- |
| `admin` | System-wide administration, accounts, catalogues and operational oversight. |
| `branch_manager` | Assigned-branch operations, permitted staff management, billing and management reports. |
| `doctor` | Own appointment roster, related patient records, consultation notes and treatments. |
| `receptionist_cashier` | Combined patient registration, scheduling and authorized billing/insurance processing. |
| `patient` | Own portal profile, appointment history, invoices and insurance information. |

Receptionist and cashier are one login role. Staff classifications do not introduce additional login roles. A role label alone does not grant unrestricted CRUD: server-side branch, ownership and relationship checks also apply. See [Roles and Reports](roles_and_reports.md).

## Functional scope

- Branch, staff, doctor, specialty, treatment and category management.
- Central patient records, emergency contacts and optional staff-provisioned portal access.
- Username-or-email sign-in, cookie-confirmed sessions and protected mutations.
- Appointment lifecycle, time-slot validation, clinical notes and multiple treatments per visit.
- Generated itemized invoices, patient receipts and separate doctor compensation.
- Patient insurance policies, per-treatment coverage and manual claims.
- Five management reports with date/branch scope and native browser print/save-as-PDF.
- PostgreSQL constraints, indexes, views, functions, procedures and triggers, plus documented migration and verification artifacts.

The frontend uses the real API; there is no browser-local database or simulated-success fallback. Source completeness does not establish successful role-scoped browser, API and database journeys.

## Deliberate boundaries

- Patients do **not** self-book, cancel or reschedule appointments in the current workflow. Public self-registration is not offered; staff provision access.
- Pharmacy stock, inpatient management, external laboratory systems, payment gateways and automated insurer integration are outside this implementation.
- Insurance uses an explicit MedSync per-policy/invoice treatment cap. Annual benefit pools, insurer settlement and historical policy-rule versioning are not modeled.
- Report PDF output is the browser's **Print / Save as PDF**, not a backend-generated PDF service.
- Secure deployment configuration, concurrent database behavior and measurable response-time targets require separate verification. They are not guarantees of the chosen stack.

The remaining release work is maintained in [Implementation Status](implementation_status.md), rather than duplicated as a separate feature backlog here.
