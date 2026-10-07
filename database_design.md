# Database Design & Guidelines - MedSync / CATMS

## Overview

The database layer is intended to support clinic appointments, patient and clinical records, billing, insurance, and analytical queries using PostgreSQL hosted on Neon.

For **CS3043 Database Systems**, the design must demonstrate relational structure, constraints, indexes, stored routines, and transaction correctness. A schema file alone does not demonstrate normalization, concurrency safety, or successful deployment.

---

## SQL Script Execution Pipeline

The numbered files in `CATMS-Backend/database/` organize implementation and evaluation evidence. They are not an automatically executed migration pipeline.

**Repository checkpoint: 2026-10-07, backend `main` at `37fcdcd`.** No Neon schema inspection or SQL execution was performed for this review.

| File | Observed repository content / Intended purpose |
| :--- | :--- |
| `01_database.sql` | Database creation and destructive table cleanup statements. Not a safe routine initialization command for the shared database. |
| `02_tables.sql` | 20 table definitions, using identity keys where applicable and a composite key for `doctor_specialty`. Current definitions need reconciliation with the submitted ER and later decisions. |
| `03_constraints.sql` | Adds the branch-manager foreign key after the mutually dependent branch/staff tables exist. |
| `04_indexes.sql` | 26 explicit `CREATE INDEX` statements. Their presence does not establish measured performance or their deployment on Neon. |
| `05_views.sql` | Header-only placeholder for view definitions/evaluation evidence. |
| `06_functions.sql` | Header-only placeholder for stored functions. |
| `07_procedures.sql` | Header-only placeholder for stored procedures. |
| `08_triggers.sql` | Header-only placeholder for triggers. |
| `09_seed_data.sql` | Seed script is present; its data, credentials, execution, and suitability for a target environment were not validated in this review. |
| `10_tests.sql` | Header-only placeholder for SQL verification evidence; no implemented assertions in this file. |

These placeholders are intentional evaluation locations. They must eventually reflect verified work, but they do not prove that corresponding objects are absent from Neon. Do not invent routine, trigger, or test behavior to fill their descriptions.

### Team Collaboration & Synchronization Workflow

1. **Central Cloud Database (Neon)**:
   Use the authorized database through backend environment configuration. Connection details and application secrets must stay outside shared documentation and frontend assets.

2. **Sharing Schema Changes**:
   Review the requirement, affected relations, API contract, existing cloud state, and safe migration before making a database change. Record confirmed DDL in the appropriate numbered file and document how it was applied. Repository and cloud state can differ; neither should silently overwrite the other.

3. **Local Container / Offline Execution**:
   The current Compose setup has no PostgreSQL container or database volume. An isolated local PostgreSQL environment would require separate setup. Do not use `docker compose down -v` as a database-reset instruction for this project.

4. **Interactive Queries & Scratch Testing**:
   Schema-only inspection can use an authorized SQL client or sanitized export. Data-changing experiments, resets, and seeds need an explicitly approved isolated target. Do not blindly execute `01_database.sql` or the entire numbered sequence on Neon.

---

## Naming Conventions & Design Standards

These are conventions for review, not a claim that every existing definition already follows them.

### 1. Identifier Conventions

- **Tables**: Singular `snake_case` nouns where consistent with the existing schema. Retain existing names such as `users_logins`; quote PostgreSQL's `"user"` identifier in SQL.
- **Columns**: Descriptive `snake_case` names, following the agreed schema and API mapping.
- **Primary Keys**: Usually `<entity>_id` identity integers; bridge relations may use a composite key.
- **Foreign Keys**: Usually `<referenced_entity>_id`, with explicit relationship semantics.

### 2. Constraint Naming

- **Foreign Keys**: Prefer `fk_<source_table>_<referenced_table>`.
- **Unique Constraints**: Prefer `uq_<table_name>_<column_name>`.
- **Indexes**: Prefer `idx_<table_name>_<column_name>`.

Names alone do not enforce a rule. Review nullability, uniqueness, valid ranges, deletion behavior, and cross-record business constraints separately.

---

## Key Data Entities & Relational Structure

The submitted ER diagram after TA feedback is a design reference. The following describes the **current repository definitions**, including changes that need explicit design reconciliation; it is not an assertion that every current column or rule was separately approved by the TA.

1. **User Authentication & Session Tracking**:
   - `user`: Account identity and password-hash storage.
   - `users_logins`: User login-event records. These are audit records, not a substitute for a stable account-to-staff relationship.

2. **Branch, Staff & Doctor Administration**:
   - `branch`: Clinic branch and manager association.
   - `staff`: Branch-assigned personnel, staff classification, and role. Its current account linkage is indirect through nullable `users_logins_id`.
   - `doctor`: Staff extension with a unique staff link and license number.
   - `specialty`: Medical specialty catalogue.
   - `doctor_specialty`: Doctor/specialty many-to-many bridge.

3. **Patient & Emergency Information**:
   - `patient`: Demographics, registration branch, and an optional unique `user_id` account association with `ON DELETE SET NULL`.
   - `emergency_contact`: Patient-associated contact details.
   - A centralized patient directory is the intended domain model. Registration branch must not be assumed to settle the final cross-branch access policy.

4. **Treatment Catalogue & Clinical Records**:
   - `treatment_category`: Service categories.
   - `treatment`: Service code, name, category, and standard price.
   - `appointment`: Patient, doctor, branch, date/time, type, creator, reschedule reference, and one optional `treatment_id`.
   - `consultation_note`: Appointment-linked clinical text and creation timestamp.
   - There is no appointment status column, doctor-availability table, or appointment/treatment bridge in the current table file. Completion/cancellation persistence and multiple clinical treatments therefore require an agreed model rather than frontend-only state.

5. **Billing, Payments & Insurance**:
   - `invoice`: One invoice per appointment, staff reference, date, declared `amount_paid`, `balance`, and status. There is no stored `total_amount` column in this definition.
   - `invoice_item`: Treatment-linked charge lines with quantity, `unitprice`, and optional description.
   - `doctor_payment`: Doctor compensation linked to an appointment and optionally an invoice item. It is not a patient receipt ledger.
   - `insurance_provider`: Provider catalogue.
   - `insurance_policy`: Patient/provider association, integer policy number, dates, and status.
   - `insurance_coverage`: Policy/treatment coverage percentage and cap.
   - `insurance_claim`: Invoice/policy association, requested amount, approved amount, date, and status.

An invoice can contain multiple charge lines; this does not create a persisted many-to-many clinical treatment relation for an appointment. Claim approval, insurer settlement, patient receipts, and doctor payouts must remain distinct. Automatic totals/balance synchronization and compensation percentages are not established by these definitions.

---

## Transaction Control & Concurrency Strategy

1. **Atomic Booking**:
   - Emergency creation and rescheduling use explicit transactions and a doctor-row lock with an overlap check in application SQL.
   - Normal appointment create/update routes do not use that same booking guard. Consistent interval validation and race-safe booking across every write path remain to be demonstrated.
   - No booking stored procedure is defined in the current procedure file. Do not describe application SQL as a deployed stored routine.

2. **Mutation Integrity**:
   - The backend pool uses autocommit. Multi-statement mutations use explicit `conn.transaction()` scopes through `database_mutation`, with rollback before known constraint errors are translated.
   - A `SELECT ... FOR UPDATE` outside an enclosing transaction does not keep a lock across later statements. Concurrency claims need verification of the whole operation, not merely the presence of a locking query.

3. **Audit & Integrity Triggers**:
   - Trigger definitions and their actual deployment/behavior remain unverified. The repository trigger file is currently a placeholder.
   - Agree the lifecycle and financial invariants before choosing which rules belong in constraints, functions/procedures, triggers, or application transactions.

---

## Database Testing & Integrity Checks

The required evidence should cover:

- Primary/foreign key, uniqueness, nullability, range, and business-rule enforcement.
- Correct transaction rollback and concurrent booking behavior.
- Reschedule history, appointment lifecycle, and authorized record access.
- Invoice-item totals, receipts/settlement, compensation, and insurance consistency.
- Stored routine/trigger behavior and report outputs against known expected results.

`10_tests.sql` does not currently implement those checks. Plan verification with isolated dummy data and a recorded target/version; do not describe historical static or simulated checks as live PostgreSQL results. See [Implementation & Integration Status](implementation_status.md) for the current evidence boundary.
