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

The final entity and attribute list must follow the latest approved SRS and ERD/TA corrections. Prototype fields or older design drafts must not be promoted to schema requirements without approval.

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
