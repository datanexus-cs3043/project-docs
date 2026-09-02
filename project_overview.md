# Project Overview - MedSync / CATMS

## Executive Summary

**MedSync / CATMS** (Clinical & Administrative Treatment Management System) is an integrated healthcare management platform designed to streamline medical channel bookings, practitioner schedules, patient clinical histories, and administrative clinic operations.

Developed under the **DataNexus** GitHub organization (`datanexus-cs3043`), this project represents the core deliverable for the **CS3043 Database Systems** module at the Department of Computer Science and Engineering, University of Moratuwa.

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

### 1. Patient Channeling & Doctor Discovery
- Search doctors by name, medical specialty, or hospital affiliation.
- Filter specialists across departments (Cardiology, General Practice, Neurology, Pediatrics, Dermatology, Dentistry, Ophthalmology).
- View doctor ratings, patient review counts, consultation fees, and available time slots.
- Real-time booking confirmation and appointment scheduling.

### 2. Clinical & Treatment Administration
- Maintain comprehensive patient profiles and medical histories.
- Log treatment plans, diagnosis notes, and prescribed interventions.
- Manage hospital affiliations and doctor shift rosters.

### 3. Database System Architecture (CS3043 Core Focus)
- Enforce strict relational constraints (Foreign keys, Unique checks, Domain validations).
- Implement performance indexes on high-throughput search columns.
- Provide database views for administrative reporting and real-time dashboard analytics.
- Enforce business logic via stored procedures, stored functions, and database triggers.
- Support atomic transactions for concurrent appointment bookings.

---

## Key Stakeholders & Roles

- **Patients**: Search for specialists, view schedules, book channel appointments, and access booking confirmations.
- **Doctors / Specialists**: View appointment rosters, manage consultation hours, and log patient clinical updates.
- **Clinic Administrators**: Manage hospital affiliation data, update specialty catalogs, oversee system-wide channeling logs, and monitor database integrity.
