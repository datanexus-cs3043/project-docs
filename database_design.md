# Database Design & Guidelines - MedSync / CATMS

## Overview

The database layer for MedSync / CATMS (`catms_db`) is engineered to support reliable, concurrent healthcare appointment transactions, patient record administration, and analytical queries.

As part of the **CS3043 Database Systems** module, the database design emphasizes relational normalization, explicit constraint definitions, performance indexing, and stored routines.

---

## SQL Script Execution Pipeline

All database scripts are maintained under `CATMS-Backend/database/` and must be executed in exact numeric sequence to satisfy dependency chains.

| Script File | Responsibility | Dependencies |
| :--- | :--- | :--- |
| `01_database.sql` | Database schema instantiation (`CREATE DATABASE catms_db`) | None |
| `02_tables.sql` | DDL table structure creation | `01_database.sql` |
| `03_constraints.sql` | Foreign key references and CHECK constraints | `02_tables.sql` |
| `04_indexes.sql` | Secondary indexes for query optimization | `02_tables.sql` |
| `05_views.sql` | Views for reporting and data abstraction | `02_tables.sql` |
| `06_functions.sql` | Deterministic and calculation scalar functions | `02_tables.sql` |
| `07_procedures.sql` | Transactional stored procedures | `03_constraints.sql`, `06_functions.sql` |
| `08_triggers.sql` | Automated audit and constraint triggers | `02_tables.sql` |
| `09_seed_data.sql` | Reference lookup data and sample test records | `03_constraints.sql` |
| `10_tests.sql` | Data integrity verification queries | `09_seed_data.sql` |

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

## Key Data Entities & Relational Structure

- **Users & Authentication**: Accounts storing system access role (`PATIENT`, `DOCTOR`, `ADMIN`).
- **Doctors & Medical Specialists**: Practitioner credentials, license numbers, specialty IDs, hospital affiliations, consultation fees, and ratings.
- **Patients**: Patient personal records, contact information, emergency contacts, and medical background.
- **Hospitals & Clinics**: Affiliated healthcare institution profiles and location details.
- **Specialties**: Master catalog of medical fields (Cardiology, Dermatology, Neurology, etc.).
- **Doctor Schedules / Channels**: Consultation windows specifying date, start time, end time, maximum patient quota, and hospital location.
- **Appointments / Bookings**: Booking transactions mapping a patient to a doctor channel slot with booking status (`PENDING`, `CONFIRMED`, `CANCELLED`, `COMPLETED`).
- **Treatments & Medical Records**: Clinical visit records, diagnosis notes, prescriptions, and follow-up directives.

---

## Transaction Control & Concurrency Strategy

1. **Atomic Channel Booking**:
   - Appointment slot reservations are executed via stored procedures (`07_procedures.sql`) using explicit transaction boundaries (`START TRANSACTION`, `COMMIT`, `ROLLBACK`).
   - Row-level locking (`SELECT ... FOR UPDATE`) is applied on schedule records to prevent double-booking during concurrent patient requests.

2. **Audit & Integrity Triggers**:
   - Triggers in `08_triggers.sql` enforce audit logging on status changes and validate appointment times against doctor availability before insert/update.

---

## Database Testing & Integrity Checks

`10_tests.sql` contains automated verification queries executed after schema setup:
- Verification of table constraint enforcement (e.g., duplicate foreign key rejection).
- Verification of stored procedure execution and transaction rollback scenarios.
- Verification of view outputs for appointment summary reports.
