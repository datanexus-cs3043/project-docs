# Database Design & Guidelines - MedSync / CATMS

## Overview

The database layer for MedSync / CATMS (`catms_db`) is engineered to support reliable, concurrent healthcare appointment transactions, patient record administration, and analytical queries.

As part of the **CS3043 Database Systems** module, the database design emphasizes relational normalization, explicit constraint definitions, performance indexing, and stored routines.

---

## SQL Script Execution Pipeline

The numbered SQL pipeline is currently an ownership and execution convention.

At the current checkpoint:

- `01_database.sql` contains database initialization.
- `02_tables.sql` through `10_tests.sql` are placeholder or skeleton files.
- Tables, constraints, indexes, views, functions, procedures, triggers, seed data, and SQL tests are still implementation work.
- No database artifact should be described as implemented until it exists and has been verified against Oracle MySQL 8.0.

### Team Collaboration & Synchronization Workflow

The SQL scripts in `CATMS-Backend/database/` serve as the **single source of truth** for all team members:

1. **Local Container Isolation**:
   Running queries or inserting data directly inside a running MySQL container (via CLI or a GUI client) only affects the developer's local machine. Local ad-hoc data is not automatically shared with teammates.

2. **Sharing Schema Changes**:
   Whenever a team member adds or updates a table, constraint, view, or procedure, the corresponding DDL/DML statements must be saved into the appropriate numbered script in `CATMS-Backend/database/` (e.g., `02_tables.sql`, `05_views.sql`) and pushed via a topic branch.

3. **Synchronizing Teammates' Environments**:
   When a teammate pulls changes from `main`, they recreate the database schema locally by running:
   ```bash
   cd CATMS-Backend
   docker compose down -v
   docker compose up --build
   ```
   The `-v` flag removes the old Docker volume so MySQL automatically re-executes all SQL scripts from `database/` in alphabetical order upon startup.

4. **Interactive Queries & Scratch Testing**:
   Developers can connect directly to their running container to test queries, inspect rows, or verify statements before committing them to the repository scripts:
   - **CLI Prompt**: `docker compose exec mysql mysql -u root -prootpassword catms_db`
   - **GUI Client (DBeaver / MySQL Workbench)**: Connect to `localhost:3306`, user `root`, password `rootpassword`, database `catms_db`.


---

## Naming Conventions & Design Standards

### 1. Identifier Conventions
- **Tables**: Plural nouns in `snake_case` (e.g., `patients`, `doctors`, `appointments`, `specialties`, `hospitals`).
- **Columns**: Singular nouns in `snake_case` (e.g., `first_name`, `created_at`, `consultation_fee`).
- **Primary Keys**: Surrogate auto-increment integer or UUID column named `id` or `<entity>_id` (e.g., `doctor_id`).
- **Foreign Keys**: Named `<referenced_entity>_id` (e.g., `specialty_id`, `hospital_id`, `patient_id`).

### 2. Constraint Naming
- **Foreign Keys**: `fk_<source_table>_<referenced_table>` (e.g., `fk_appointments_doctors`).
- **Unique Constraints**: `uq_<table_name>_<column_name>` (e.g., `uq_doctors_license_number`).
- **Indexes**: `idx_<table_name>_<column_name>` (e.g., `idx_appointments_date_status`).

---

## Key Data Entities & Relational Structure (Approved ER Specification)

The schema adheres to the TA-approved Entity-Relationship model (`external-docs/ER submission-2final_touch.pdf`) comprising 20 normalized relations:

1. **User Authentication & Session Tracking**:
   - `user`: System user accounts, credentials, contact details.
   - `users_logins`: Session and access logs linking users to their login timestamps.

2. **Branch, Staff & Doctor Administration**:
   - `branch`: Multi-specialty clinic facilities (Colombo, Kandy, Galle), address, contact details, and branch manager association.
   - `staff`: Medical and non-medical employees assigned to branches, linked to user credentials with specific functional roles.
   - `doctor`: Clinical practitioner extension linked to staff records, storing license numbers and doctor names.
   - `specialty`: Master medical disciplines catalog (e.g., General Medicine, ENT, Paediatrics, Cardiology).
   - `doctor_specialty`: Many-to-many bridge linking doctors to one or more practicing specialties.

3. **Patient & Emergency Information**:
   - `patient`: Centralized patient directory with demographic data, cross-branch registration, and unique patient identifiers.
   - `emergency_contact`: Designated emergency contacts for patients with relationship and telephone details.

4. **Treatment Catalogue & Clinical Records**:
   - `treatment_category`: Broad service classifications (Consultations, Diagnostics, Procedures, Injections).
   - `treatment`: Predefined medical service catalog with standardized service codes, names, and base prices.
   - `appointment`: Core channeling and walk-in consultation records specifying patient, doctor, branch, date, time slots, appointment type, and reschedule links.
   - `consultation_note`: Post-appointment clinical observations, symptoms, diagnosis, and medical advice.

5. **Billing, Payments & Insurance**:
   - `invoice`: Financial statements generated upon completed appointments, tracking total charges, amount paid, and outstanding balances.
   - `invoice_item`: Line items capturing treatments rendered, unit prices, and quantities.
   - `doctor_payment`: Practitioner compensation and remuneration disbursements linked to appointments and billed services.
   - `insurance_provider`: Master catalog of registered health insurance companies.
   - `insurance_policy`: Patient-held insurance policies with coverage terms, validity intervals, and policy numbers.
   - `insurance_coverage`: Policy-specific coverage rules, reimbursement percentages, and maximum allowable caps per treatment.
   - `insurance_claim`: Claims filed against invoices, recording requested reimbursements, approved amounts, and claim status.

---

## Transaction Control & Concurrency Strategy

1. **Atomic Channel Booking**:
   - The approved design requires atomic appointment operations and appropriate transaction handling. The stored procedures, locking strategy, and verification tests must be implemented and validated before this behavior is described as available.
   - Row-level locking (`SELECT ... FOR UPDATE`) is applied on schedule records to prevent double-booking during concurrent patient requests.

2. **Audit & Integrity Triggers**:
   - Triggers in `08_triggers.sql` enforce audit logging on status changes and validate appointment times against doctor availability before insert/update.

---

## Database Testing & Integrity Checks

`10_tests.sql` contains automated verification queries executed after schema setup:
- Verification of table constraint enforcement (e.g., duplicate foreign key rejection).
- Verification of stored procedure execution and transaction rollback scenarios.
- Verification of view outputs for appointment summary reports.
