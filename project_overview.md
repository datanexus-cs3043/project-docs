# Project Overview - MedSync / CATMS

## Executive Summary

**MedSync / CATMS** is a database-centered clinic management system for CS3043. It is intended to manage clinic branches, staff and doctors, patient records, appointments, consultations and treatments, invoices, insurance information, users, and required management reports.

The current React application is an early visual prototype. Its hard-coded doctors, ratings, hospitals, specialties, and booking alert demonstrate interface ideas only; they are not automatically approved system requirements.

---

## Academic Context

- **Institution**: University of Moratuwa, Sri Lanka
- **Department**: Department of Computer Science and Engineering
- **Module**: CS3043 - Database Systems
- **Organization**: DataNexus (`datanexus-cs3043`)
- **Team Size**: 5 Undergraduate Software Engineering / Computer Science Students

---

## Core Problem Statement

Healthcare facilities and outpatient channeling centers frequently encounter operational bottlenecks in:

1. **Appointment Scheduling**: Overbooking, time slot conflicts, and inefficient queue management.
2. **Channel Transparency**: Difficulty for patients in finding verified specialists across multiple hospitals and consultation windows.
3. **Data Integrity & Consistency**: Managing high-concurrency appointment updates, patient records, and treatment logs while preserving relational integrity.
4. **Administrative Overhead**: Manual record-keeping for doctor availabilities, hospital affiliations, and medical records.

MedSync / CATMS resolves these challenges by coupling an intuitive React frontend with a high-performance Spring Boot backend and an optimized MySQL relational database schema.

---

## System Scope & Functional Requirements

### 1. Clinic and Patient Management

- Manage clinic branches, staff, doctors, and specialties.
- Register and maintain centralized patient records.
- Maintain emergency-contact and insurance-policy information.
- Support the approved appointment lifecycle, including creation, cancellation, rescheduling, status changes, and completion.
- Record consultation and treatment information according to the approved ERD.

### 2. Billing, Insurance, and Reporting

- Maintain invoices and payment-summary information.
- Support insurance-related information and claims according to the approved design.
- Provide the required management reports.

### 3. Database-System Requirements

- Implement the approved relational schema.
- Enforce primary keys, foreign keys, uniqueness, domain, and business constraints.
- Add indexes, views, stored functions, procedures, triggers, seed data, and SQL tests where required.
- Demonstrate transaction correctness and concurrency handling where required.
- Validate the final implementation against Oracle MySQL 8.0/InnoDB.

---

## Key Stakeholders & Roles

- **Patients**: Search for specialists, view schedules, book channel appointments, and access booking confirmations.
- **Doctors / Specialists**: View appointment rosters, manage consultation hours, and log patient clinical updates.
- **Clinic Administrators**: Manage hospital affiliation data, update specialty catalogs, oversee system-wide channeling logs, and monitor database integrity.
