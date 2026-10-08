# Project Overview - MedSync / CATMS

## Executive Summary

**MedSync / CATMS** is a database-centered clinic management system for CS3043. It is intended to manage clinic branches, staff and doctors, centralized patient records, appointments, consultations and treatments, invoices, insurance information, users, and management reports.

The current implementation uses **React/TypeScript, FastAPI, and PostgreSQL hosted on Neon**. The frontend contains domain pages and API service wrappers; the former local simulator/demo fallback has been removed. Authentication and selected doctor workflows have received integration corrections, while patient self-service, report contracts and other workflows remain incomplete. See [Implementation & Integration Status](implementation_status.md) for the dated checkpoint.

---

## Academic Context

- **Institution**: University of Moratuwa, Sri Lanka
- **Department**: Department of Computer Science and Engineering
- **Module**: CS3043 - Database Systems
- **Organization**: DataNexus (`datanexus-cs3043`)
- **Team Size**: 5 students

The module requirements, submitted SRS, final submitted ER diagram and TA feedback guide the scope. Later stack changes and team decisions must be recorded separately. The current repository schema must not be described as fully TA-approved merely because it contains 20 tables.

---

## Core Problem Statement

Clinic branches and outpatient channeling services need consistent handling of:

1. **Appointment Scheduling**: Booking, rescheduling, cancellations, and time-slot conflicts.
2. **Patient Information**: A centralized patient record usable across clinic branches with appropriate access controls.
3. **Clinical & Financial Integrity**: Linking consultations, treatments, invoice items, insurance claims, and doctor compensation without confusing their meanings.
4. **Administrative Reporting**: Reliable appointment, revenue, outstanding-balance, treatment, and insurance summaries.

MedSync / CATMS aims to address these needs through a shared relational database and role-specific workflows. Concurrency safety, performance, and correct financial reconciliation are outcomes to demonstrate, not guarantees established by the stack alone.

---

## System Scope & Functional Requirements

The following describes intended project scope, not a checklist of delivered features.

### 1. Clinic and Patient Management

- Manage clinic branches, staff, doctors, and specialties.
- Register and maintain centralized patient records.
- Maintain emergency-contact and insurance-policy information.
- Support the required appointment lifecycle: creation, cancellation, rescheduling, status changes, and completion.
- Record consultation and treatment information in an agreed relational model.

### 2. Billing, Insurance, and Reporting

- Maintain invoices, itemized charges, payment summaries, and doctor compensation.
- Support insurance providers, patient policies, treatment coverage, and claims.
- Provide management reports for appointment summaries, doctor revenue, outstanding balances, treatments by category, and insurance versus out-of-pocket amounts.
- Generate the necessary reports as PDFs for viewing and download, as clarified on 2026-10-08. This is deferred delivery work; no PDF implementation is claimed. Data definitions and authorized scopes must be settled before formatting/export.
- Define receipt, claim-approval, settlement, and compensation rules explicitly. They are separate financial events; the current payment API records doctor compensation, not patient receipts.

### 3. Database-System Requirements

- Implement and reconcile the relational schema with the submitted design and confirmed changes.
- Enforce primary keys, foreign keys, uniqueness, domain, and business constraints.
- Provide indexes, views, stored functions, procedures, triggers, seed data, and SQL verification evidence where required.
- Demonstrate atomic transactions and concurrency handling for relevant operations.
- Retain the numbered SQL files as an accessible evaluation record. Views/functions/procedures/triggers and schema extensions are now present in source; their presence does not establish deployment or API integration. The SQL verification file remains a placeholder.
- Target PostgreSQL compatibility as described in the repository SQL. The actual cloud version and deployed objects require separate verification.

### Scope Boundaries

This project concerns the clinic's branches, not integration with unrelated hospitals. Pharmacy stock, inpatient management, external laboratory/insurer integrations, and payment gateways are not part of the recorded core scope. Interface mock data and simulator rules do not add approved requirements.

---

## Key Stakeholders & Roles

The application declares **five login roles**, with receptionist and cashier combined:

| Role | Intended responsibility |
| :--- | :--- |
| `admin` | System-wide administration, catalogues, accounts, and oversight. |
| `branch_manager` | Branch operations, staff oversight, and management reporting within the agreed branch scope. |
| `doctor` | Authorized appointment rosters, patient clinical information, and consultation records. |
| `receptionist_cashier` | Registration, appointment coordination, and authorized billing operations. |
| `patient` | Access to their own authorized appointment, profile, and billing information. |

These responsibilities are a domain overview, not a grant of unrestricted operations. [Roles, Patient Self-Service & Report Output](roles_and_reports.md) consolidates the October 8 reference into five roles and records source differences. Patient is already declared; its remaining functionality is deferred, not being added as a sixth role. Self-service booking, cross-branch access, provisioning, clinical-note access and doctor schedule/profile editing require agreed policy and API support. A staff classification such as nurse does not create another login role.
