# Database Design & Guidelines - MedSync / CATMS

## Overview

The database layer is intended to support clinic appointments, patient and clinical records, billing, insurance, and analytical queries using PostgreSQL hosted on Neon.

For **CS3043 Database Systems**, the design must demonstrate relational structure, constraints, indexes, stored routines, and transaction correctness. A schema file alone does not demonstrate normalization, concurrency safety, or successful deployment.

---

## SQL Script Execution Pipeline

The numbered files in `CATMS-Backend/database/` organize implementation and evaluation evidence. They are not an automatically executed migration pipeline.

**Repository checkpoint: 2026-10-08, backend `main` at `1405700`.** No Neon schema inspection or SQL execution was performed for this documentation review. The October 7 placeholder-only description of files 05–08 is superseded.

| File | Observed repository content / Intended purpose |
| :--- | :--- |
| `01_database.sql` | Database creation and destructive table cleanup statements. Not a safe routine initialization command for the shared database. |
| `02_tables.sql` | 20 base table definitions, using identity keys where applicable and a composite key for `doctor_specialty`. This is not the entire schema: later files add status and two tables. Reconcile later changes with the submitted ER and recorded decisions. |
| `03_constraints.sql` | Adds the branch-manager foreign key after the mutually dependent branch/staff tables exist. |
| `04_indexes.sql` | 26 ordinary index statements plus a normalized specialty-name unique index on `LOWER(BTRIM(specialty_name))`. Presence does not establish measured performance or deployment on Neon. |
| `05_views.sql` | Drops/redefines views, adds appointment status and its check, creates `appointment_treatment` and `patient_payment` with indexes/checks, then defines six views. This is schema-changing SQL, not just a report query. |
| `06_functions.sql` | Eleven function definitions covering invoice recalculation/locking, receipt synchronization, appointment/treatment rules, insurance claims and doctor compensation. |
| `07_procedures.sql` | Four procedures: `sp_record_patient_payment`, `sp_complete_appointment`, `sp_cancel_appointment` and `sp_reschedule_appointment`. |
| `08_triggers.sql` | `btree_gist` extension, doctor/time exclusion constraint, financial/domain checks and eleven triggers using the functions above. |
| `09_seed_data.sql` | Seed script is present; its data, credentials, execution, and suitability for a target environment were not validated in this review. |
| `10_tests.sql` | Header-only placeholder for SQL verification evidence; no implemented assertions in this file. |
| `migrations/20261008_specialty_name_uniqueness.sql` | Separate transaction-scoped migration for the normalized specialty-name index, with lock/statement timeouts. Committed in `ff9a26e`; Neon application is unknown. |

The numbered files are evaluation locations, not interchangeable deployment commands. Source definitions now exist for routines/reporting and several extensions, while `10_tests.sql` is still a placeholder. Neither definitions nor placeholders establish the actual Neon state. Do not duplicate existing objects from a base-file-only reading or invent verification evidence.

### Extensions and Reporting Views

The six defined views are `v_branch_daily_appointment_summary`, `v_invoice_totals`, `v_doctor_revenue`, `v_outstanding_balances`, `v_treatment_category_usage` and `v_insurance_vs_out_of_pocket`. They depend on base and extension objects. Current report API handlers still query base tables rather than these views, so their result definitions are not automatically aligned.

Appointment status and patient receipts are present in repository SQL but not consistently exposed by the existing APIs. In particular, complete/cancel handlers still return 501, treatment endpoints retain the single-FK contract, and `/payments` endpoints still operate on doctor compensation. Verify the selected API/schema contract before proposing more tables or claiming feature completion.

### Team Collaboration & Synchronization Workflow

1. **Central Cloud Database (Neon)**:
   Use the authorized database through backend environment configuration. Connection details and application secrets must stay outside shared documentation and frontend assets.

2. **Sharing Schema Changes**:
   Review the requirement, affected relations, API contract, existing cloud state and safe migration before making a database change. Record confirmed DDL and how it was applied. Repository and cloud state can differ; neither should silently overwrite the other. Check dependencies and existing data before applying extensions, exclusion constraints or uniqueness. An existing migration is not permission to run it.

3. **Local Container / Offline Execution**:
   The current Compose setup has no PostgreSQL container or database volume. An isolated local PostgreSQL environment would require separate setup. Do not use `docker compose down -v` as a database-reset instruction for this project.

4. **Interactive Queries & Scratch Testing**:
   Schema-only inspection can use an authorized SQL client or sanitized export. Data-changing experiments, resets and seeds need an explicitly approved isolated target. Do not blindly execute `01_database.sql` or the entire numbered sequence on Neon. `05_views.sql` also contains view drops and schema changes; its filename does not make it a safe read-only operation.

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
   - `appointment`: The base file has patient, doctor, branch, date/time, type, creator, reschedule reference and one optional `treatment_id`; file 05 adds Scheduled/Completed/Cancelled status.
   - `consultation_note`: Appointment-linked clinical text and creation timestamp.
   - `appointment_treatment`: File 05 adds the clinical bridge with unique appointment/treatment pairs, positive quantity, nonnegative unit price and at most one primary treatment. The legacy appointment FK and API contract need reconciliation with this relation.
   - No dedicated doctor-availability table was found in these definitions. Persisted lifecycle and multiple-treatment definitions exist in SQL, but deployment and consistent API/UI use remain to be established.

5. **Billing, Payments & Insurance**:
   - `invoice`: One invoice per appointment, staff reference, date, declared `amount_paid`, `balance`, and status. There is no stored `total_amount` column in this definition.
   - `invoice_item`: Treatment-linked charge lines with quantity, `unitprice`, and optional description.
   - `doctor_payment`: Doctor compensation linked to an appointment and optionally an invoice item. It is not a patient receipt ledger.
   - `patient_payment`: File 05 adds a separate invoice-linked ledger with signed amounts, payment/refund/adjustment types and receiving-staff attribution. It is not what the legacy `/payments` APIs currently write.
   - `insurance_provider`: Provider catalogue.
   - `insurance_policy`: Patient/provider association, integer policy number, dates, and status.
   - `insurance_coverage`: Policy/treatment coverage percentage and cap.
   - `insurance_claim`: Invoice/policy association, requested amount, approved amount, date, and status.

Invoice charge lines and clinical treatment records remain distinct even though a clinical bridge now exists. Claim approval, insurer settlement, patient receipts and doctor payouts must also remain distinct. Functions/triggers define financial synchronization behavior in source, but application-wide correctness, deployed behavior and any compensation formula still need evidence and agreed business meaning.

---

## Transaction Control & Concurrency Strategy

1. **Atomic Booking**:
   - Emergency creation and rescheduling use explicit transactions and a doctor-row lock with an overlap check in application SQL.
   - Normal appointment create/update routes do not use that same application booking guard. File 08 now defines a doctor/time-range exclusion constraint for non-cancelled appointments and appointment-validation triggers; deployment and consistent error handling across every write path remain unverified.
   - Reschedule/completion/cancellation procedures are present in file 07. Their existence is not proof the API calls them or that they are deployed; preserve the distinction between application SQL, routine definitions and runtime behavior.

2. **Mutation Integrity**:
   - The backend pool uses autocommit. Multi-statement mutations use explicit `conn.transaction()` scopes through `database_mutation`, with rollback before known constraint errors are translated.
   - A `SELECT ... FOR UPDATE` outside an enclosing transaction does not keep a lock across later statements. Concurrency claims need verification of the whole operation, not merely the presence of a locking query.

3. **Audit & Integrity Triggers**:
   - Trigger definitions now cover appointment validation/treatment synchronization, invoice preparation/recalculation, patient-payment synchronization, completion and claim/compensation validation. Actual deployment and whole-workflow behavior remain unverified.
   - Compare those existing rules with the agreed lifecycle/financial invariants before adding or replacing enforcement. Avoid two divergent definitions of the same rule in API and SQL.

---

## Database Testing & Integrity Checks

The required evidence should cover:

- Primary/foreign key, uniqueness, nullability, range, and business-rule enforcement.
- Correct transaction rollback and concurrent booking behavior.
- Reschedule history, appointment lifecycle, and authorized record access.
- Invoice-item totals, receipts/settlement, compensation, and insurance consistency.
- Stored routine/trigger behavior and report outputs against known expected results.

`10_tests.sql` does not currently implement those checks. Earlier isolated PostgreSQL 16.15 work validated selected procedures and later specialty mutation/migration cases, not the complete current SQL pipeline, Neon deployment or all application workflows. This documentation pass did not execute SQL. Future approved verification should use isolated dummy data and a recorded source/target/version; see [Implementation & Integration Status](implementation_status.md) for provenance.

Reports must have agreed grain, period, status and financial semantics before generating PDFs. Source views or a valid PDF file alone do not establish correct totals. See [Roles, Patient Self-Service & Report Output](roles_and_reports.md) for the deferred output requirement.
