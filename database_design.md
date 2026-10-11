# Database Design & Guidelines — MedSync / CATMS

## Baseline and design intent

Source reviewed **12 October 2026**, backend `development@4f22ab4`. The numbered SQL defines **22 tables, six views, eleven functions, five procedures and eleven triggers**, including extensions after the 20-table base.

The database separates identity, branch operations, clinical activity and financial events. Constraints, stored routines and transactions complement API checks; they do not replace application authorization. The submitted ER diagram remains a design reference requiring reconciliation with later extensions.

## Relation map

| Domain | Relations | Important relationships |
| --- | --- | --- |
| Identity | `"user"`, `users_logins` | Accounts and login audit. Patient links directly to an optional unique `user_id`; staff account resolution uses its existing login-audit association. This legacy staff link is not a direct staff-account FK. |
| Organization | `branch`, `staff`, `doctor`, `specialty`, `doctor_specialty` | Staff belong to branches; doctor extends staff. Doctor/specialty is many-to-many. Branch manager assignment creates a branch/staff dependency. |
| Patients | `patient`, `emergency_contact` | Registration branch, demographics, optional account and multiple contacts. Access also depends on patient ownership or the doctor's appointment relationship. |
| Care | `treatment_category`, `treatment`, `appointment`, `appointment_treatment`, `consultation_note` | Visits link patient/doctor/branch; multiple treatment rows store quantity and captured unit price; notes link to visits. |
| Billing | `invoice`, `invoice_item`, `patient_payment`, `doctor_payment` | One invoice per visit. Itemized charges, signed patient ledger and doctor compensation are separate relations. |
| Insurance | `insurance_provider`, `insurance_policy`, `insurance_coverage`, `insurance_claim` | Patient policies, treatment-specific coverage and invoice/policy claims. Approval is not cash settlement. |

Identifiers generally use singular `snake_case`; retain established names such as `users_logins`. Quote PostgreSQL's `"user"` table. Most keys are identity integers; `doctor_specialty` uses a composite key.

### Modeling decisions

- Separate provider/policy/coverage relations prevent repeating provider details on every claim; treatment categories and specialties are catalogues rather than free-text copies on every visit.
- Doctor/specialty and appointment/treatment bridges represent multi-valued relationships without adding repeated columns to their parent records.
- Captured treatment/invoice prices are intentional historical snapshots, not redundant copies to remove when the live catalogue changes.
- Invoice paid amount, balance and status are stored derived values maintained by financial routines/triggers. This makes correct synchronization essential; they are not independent user-entered facts.
- Lookup indexes support common branch, doctor, patient and invoice joins; normalized specialty uniqueness enforces a data rule as well as providing a lookup structure. The doctor/time exclusion constraint is an integrity mechanism, not merely a performance index.
- The legacy staff/login-audit association and mutable coverage rules remain design limitations. This document does not certify every relation as fully normalized or every historical business rule as versioned.

### Clinical and billing boundaries

- `appointment.status` is Scheduled, Completed or Cancelled. Rescheduling preserves `original_appointment_id`.
- `appointment_treatment` holds unique appointment/treatment pairs, positive quantities, nonnegative captured prices and at most one primary treatment. The legacy `appointment.treatment_id` is retained and synchronized with the primary service.
- Changing a catalogue price does not reprice previously captured visit treatments or issued items.
- Completion requires a treatment or consultation evidence; generation additionally requires a completed appointment with a treatment-bridge row. A note-only completed visit cannot be invoiced until a treatment is recorded.
- `sp_generate_invoice` copies recorded quantities/prices into invoice items and prevents a second invoice for the same visit. Issued items are immutable through the application item-write endpoints.
- SQL uses `unitprice`; the appointment-treatment API exposes `unit_price`. Do not silently rename persisted columns to match a client field.

### Financial meanings

| Quantity/event | Meaning |
| --- | --- |
| Billed amount | Sum of invoice-item quantity × unit price. There is no stored `invoice.total_amount` column in the baseline. |
| Patient paid | Net signed `patient_payment` ledger; payment/refund/adjustment rows are distinct types. |
| Balance and status | Recalculated from charges and the patient ledger; API callers cannot directly replace them. |
| Doctor compensation | `doctor_payment`; not patient money received and not the doctor-revenue report's billed total. |
| Claim approval | Manual approval of an insurance request; does not create a receipt, settle insurer cash or automatically reduce invoice balance. |

The public receipt API records positive payments through `sp_record_patient_payment`. Ledger support for refund/adjustment types is **not** a claim that a dedicated correction UI/API is implemented. An uncertain payment outcome must be reconciled before recording another receipt.

Insurance follows the explicit per-policy/invoice rules in [Roles and Reports](roles_and_reports.md). Annual entitlement, settlement and coverage-rule versioning are absent. The newest trigger migration is a release prerequisite, not an already installed rule.

## SQL artifact organization

These are review/setup artifacts, **not** an automatically executed deployment pipeline.

| File | Content and operational caution |
| --- | --- |
| `01_database.sql` | Database creation and destructive table cleanup. Never run on the populated shared database as routine setup. |
| `02_tables.sql` | Twenty base tables, keys and checks; not the complete extended schema. |
| `03_constraints.sql` | Adds the branch-manager relationship after mutually dependent tables exist. |
| `04_indexes.sql` | Foreign-key/query lookup indexes and normalized specialty-name uniqueness using `LOWER(BTRIM(specialty_name))`. |
| `05_views.sql` | Appointment status, treatment bridge, receipt ledger, supporting constraints/indexes and six views. Contains view drops; not a read-only report script. |
| `06_functions.sql` | Eleven functions for financial synchronization, appointment/treatment validation and claim/compensation rules. |
| `07_procedures.sql` | `sp_record_patient_payment`, `sp_complete_appointment`, `sp_cancel_appointment`, `sp_reschedule_appointment`, `sp_generate_invoice`. |
| `08_triggers.sql` | Eleven triggers, domain checks and `btree_gist` doctor/time exclusion enforcement. |
| `09_seed_data.sql` | Fixed-ID sample data. Do not replay into an existing shared dataset or overwrite current accounts. |
| `10_tests.sql` | Implemented assertions for lifecycle/schema readiness, sample data, treatment synchronization, ledger reconciliation, overlaps and required views. Not a placeholder and not proof it passed on a given target. |

The six views are `v_branch_daily_appointment_summary`, `v_invoice_totals`, `v_doctor_revenue`, `v_outstanding_balances`, `v_treatment_category_usage` and `v_insurance_vs_out_of_pocket`. The API uses the branch-summary and outstanding views; other report routes use their own queries. Matching view and report names does not establish identical semantics.

### Forward migrations

| Migration | Purpose | Current evidence boundary |
| --- | --- | --- |
| `20261008_specialty_name_uniqueness.sql` | Normalized specialty-name uniqueness. | Existing source migration; target installation must be checked before replay. |
| `20261010_generate_invoice.sql` | Completed-visit invoice generation from captured treatments. | Existing source migration; verify target procedure definition. |
| `20261012_insurance_claim_coverage.sql` | Replaces claim-validation function/trigger for coverage, reservations, immutable invoice/policy identity and legacy rejection handling. | **Committed, unapplied and not PostgreSQL-compiled at this checkpoint.** |

The latest migration changes a function/trigger only; it does not delete, backfill or rewrite patient/financial records. Existing numbered claim definitions differ: replaying old functions/triggers after this migration would undo its rules. Apply approved migrations chronologically after their required baseline, on a separately authorized target.

A recorded **11 October QA reconciliation** established the extended schema and added synthetic linked data while preserving existing records. It predates the newest insurance migration and is not a fresh cloud inspection or a complete browser/API acceptance result. Account details and connection credentials are not part of this repository.

## Transactions and concurrency

- Backend mutations explicitly scope transactions around multi-statement work. Exceptions must roll back before translating known database errors into API responses.
- Booking checks doctor/branch relationships and overlaps; the database exclusion constraint protects overlapping non-cancelled doctor slots when installed. Verify competing sessions, not only sequential requests.
- Registration creates account, patient and optional contact atomically. Account writers serialize cross-column case-insensitive username/email checks; legacy ambiguous accounts are not automatically repaired.
- Invoice/payment mutations lock the financial parent and recalculate derived values through routines/triggers. Verify partial payment, overpayment rejection and rollback on the installed PostgreSQL definitions.
- The new insurance migration requires **Read Committed**. Direct claim-update writers must pre-lock the invoice before the claim row; coverage writers must lock their parent policy before changing rules. Unsupported isolation is rejected. Actual PostgreSQL concurrent-session behavior remains to be validated.
- Direct SQL bypasses API role checks. Administrative access requires its own controls; triggers are not an application permission system.

Indexes and request coalescing are implemented mechanisms, not measured performance gains. Collect representative `EXPLAIN (ANALYZE, BUFFERS)` results on an approved target before claiming tuning effectiveness.

## Safe database change process

1. Confirm the exact authorized environment, current objects and existing data; use sanitized schema evidence rather than sharing credentials.
2. Review dependencies, invariant changes, API/UI impact and migration ordering.
3. Validate forward-only changes and relevant concurrency cases on an isolated PostgreSQL target first.
4. Obtain target-specific approval, a recovery plan and an application window before shared writes.
5. Record applied migration/version, post-change object checks and financial/lifecycle reconciliation.
6. Preserve existing rows and account passwords. Do not reset/reseed, blindly run the numbered sequence or use an arbitrary rollback wrapper as proof of safety.

Compose does not provision PostgreSQL. Further migration execution and shared data changes are separate operations from updating these documents.
