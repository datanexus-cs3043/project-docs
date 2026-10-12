# Original assignment coverage

Reviewed 12 October 2026 against both applications' `development` working trees.
Authority: **Project 1 - Clinic Appointment and Treatment Management System**, the
original two-page assignment. Page 1 defines the clinic workflows; page 2 defines
five reports and the database-design task. Useful additional features are retained.
This is a source/verification map, not a certificate that hosted main has passed QA.

## Functional requirements

| ID | Original requirement | Current implementation and correction | Acceptance boundary |
| --- | --- | --- | --- |
| F01 | Branch manager plus medical/non-medical staff at each branch | Branch/staff relations and management pages; administrators can now assign an own-branch manager from the Branches page. API validates the manager role, not just the staff FK. SQL also protects assignments and historical staff transfers. | A new branch can be provisioned before staff exist; assign its manager before operational acceptance. Fresh seed has a valid manager at every branch. New SQL guards require the forward migration on an existing target. |
| F02 | Doctor at a specific branch, with multiple specialties | Doctor links to one staff/branch record; `doctor_specialty` is a many-to-many bridge with a composite PK. Existing doctor-profile specialty controls remain. Booking validates doctor/branch; the new SQL guard covers direct booking writes too. | Fresh seed now includes two multi-specialty physicians. Transfers with existing clinical/financial history require a separate history model rather than rewriting the branch. |
| F03 | Register at any branch; access centralized personal/emergency/insurance records across branches | Staff register at their own branch, administrators at any branch. Authorized front desk/managers can read the shared patient directory/details, emergency contacts and insurance information regardless of registration branch. Doctors require a related own-branch appointment; patients require ownership. | Shared read access does not grant another branch's demographic/policy editing or unrestricted clinical-note access. Validate the visiting-branch browser journey. |
| F04 | Patient/doctor time slots; no doctor overlaps; Scheduled/Completed/Cancelled | Validated local date/time request models; lifecycle handlers/procedures; doctor/time GiST exclusion constraint excludes Cancelled rows and uses half-open intervals so adjacent slots are allowed. | Two competing inserts into the same doctor/slot were checked in isolated PostgreSQL: exactly one committed. This is not a load benchmark or hosted-concurrency certification. |
| F05 | Completed visits with a treatment set and/or consultation notes | Consultation notes and multiple captured-price treatment lines; doctor-own clinical scope; completion requires recorded evidence. A note-only visit can complete, then have more notes/catalogue services recorded until invoicing. Cancelled/invoiced edit guards remain. | Evidence-before-completion is an explicit workflow assumption; recording after completion is also supported, as the brief describes. An unpriced note alone is not silently billed as a fee. |
| F06 | Catalogue service code and price; invoice completed care | Unique service codes, nonnegative finite monetary values, quantities and captured prices; `sp_generate_invoice` creates one itemized invoice per Completed visit in one transaction. Direct-SQL invoice creation now validates completion and staff branch. | Catalogue edits do not reprice recorded care. Generating a note-only invoice requires adding the actual consultation/service charge first. New direct-SQL guards remain unapplied on shared Neon. |
| F07 | Full/partial patient payments and outstanding dues | Separate signed `patient_payment` ledger, locked payment procedure, no-overpayment checks, derived balance/status triggers and payment/history pages. Doctor compensation is separate. | Existing payment outcome/refresh safeguards remain. Browser full/partial/rejected-payment reconciliation is still required; there is no automatic exactly-once delivery claim. |
| F08 | Partial/full insurance benefits for selected services under policy terms | Provider/policy/coverage/claim model; date/ownership/amount eligibility; grouped service percentages/caps; Pending reservations, Approved decisions and Rejected release. Fresh seed includes illustrative 80% and 100% coverage. | Per-policy/invoice caps are documented assumptions, not insurer contracts. Approval is not cash settlement or a patient receipt. Annual limits and insurer integration remain outside this phase. |
| F09 | Rescheduling | Locked procedure creates a successor and cancels its predecessor in one transaction; API exposes both history directions. Ordinary booking updates now reject direct date/time overwrites. | History survives unsuccessful rescheduling. Fresh seed includes a rescheduled visit; verify navigation in the running application. |
| F10 | Staff-created emergency walk-ins without prior booking | Existing staff walk-in action and emergency route create a same-day time-slotted visit; doctor/branch and overlap checks still apply. Controls no longer guess branch 1 or request a forbidden branch list for doctors. | Emergency means direct staff creation, not bypassing the no-overlap rule or enabling patient self-booking. Midnight-crossing same-day slots are rejected. |
| R01 | Daily branch appointments by status | Daily branch view and `/reports/appointments-summary`; page supports one day or inclusive period plus branch, with all three status counts. | Empty days are not synthesized into calendar rows. Compare known daily totals in the browser. |
| R02 | Doctor-wise revenue | `/reports/doctor-revenue` groups invoiced visit value and net patient receipts by doctor and branch ID. Same-name branches are not merged. | Billed service revenue is not doctor compensation. Date filtering selects invoice issue dates and reports their current values. |
| R03 | Patients with outstanding balances | `/reports/outstanding-balances` aggregates positive invoice balances, including partially paid invoices, by patient within the selected branch/period. | The oldest invoice belongs to the selected cohort, not necessarily the patient's lifetime debt. |
| R04 | Treatment count by category over a period | `/reports/treatments-by-category` and the corrected SQL usage view count quantities on Completed visits; use the view's appended `appointment_date` for period filters. Page/export shows category subtotals as well as procedure detail. | Completed uninvoiced care is included. Captured service value is not cash collected. |
| R05 | Insurance coverage vs out-of-pocket payments | `/reports/insurance-vs-out-of-pocket` separates Approved coverage decisions, net patient receipts, outstanding balances and estimated patient exposure. Claims are aggregated before invoice totals to avoid multiplication. | Approved coverage is not settled insurer cash. The page labels this distinction and exports the loaded branch/date scope. |

## Database and non-functional necessities

| ID | Requirement or necessity | Mechanism / evidence | What is not yet proved |
| --- | --- | --- | --- |
| N01 | Model entities and relationships; PK/FK consistency | 22 relations, generated identifiers, FK relationships, unique invoice-per-visit and bridge keys; normalized catalogues, policy/coverage separation and documented price snapshots. | No blanket claim that every legacy relationship is fully normalized; staff's login-audit linkage remains a design limitation. |
| N02 | Procedures/functions/triggers supporting ACID | Transactional visit changes, invoice generation and receipts; parent-row locks; database constraints/exclusion; derived balance triggers; failed operations roll back. Fresh PostgreSQL setup and a failing multi-step transaction were exercised locally. | Complete concurrent payment/claim/coverage/administration acceptance and durability/recovery drills are not established by synthetic handler checks. |
| N03 | Appropriate indexing | FK/join indexes, branch/date and active doctor/date lookups, normalized specialty uniqueness, invoice-date lookups and new case-insensitive username/email expression indexes. The overlap exclusion is an integrity mechanism. | Index presence is not a measured response-time improvement. Measure representative query plans/data sizes before claiming speed or capacity. |
| N04 | Domain understanding and explicit assumptions | `database/SEED_DATA.md` documents Sri Lankan naming/contact conventions, public reference sources, fictional people/contacts/licences, illustrative LKR prices and coverage assumptions. Financial/report definitions are in Roles and Reports. | No copied real patient histories, asserted real licences or regulatory/product-compliance claim. |
| N05 | Dummy data before presentation | Guarded empty-schema seed fills every table and demonstrates three branches, staff types, cross-branch care, multi-specialty doctors, all visit states, multiple treatments, rescheduling, invoices, receipts and claim outcomes. | A fresh setup was validated locally. Do not replay it over shared data. Existing shared demonstration data belongs to the separately recorded maintenance operation. |
| N06 | UI for QA of key features | Registration, directory, booking/walk-in, clinical records, completion, invoicing, payment, insurance and five report pages; server authorization and visible errors/confirmations remain authoritative. | Real role-scoped browser journeys, keyboard/accessibility, responsive layouts and native print/PDF visual checks remain acceptance work. The brief does not require a dedicated dummy-data entry UI. |
| N07 | Correct clinic dates and safe operation | API pool connections use Asia/Colombo; frontend daily filters and walk-in clocks use the same timezone. Invalid inputs fail validation rather than silently selecting defaults; read-only treatment/policy/coverage reads avoid unnecessary row locks. | Verify the selected runtime rebuild, timezone at clinic midnight and actual cross-origin session settings. |

## SRS additions and operational targets

The original brief **does not state** numeric latency, concurrency, uptime,
inactivity-timeout or backup targets. The team SRS adds them; preserve that origin
instead of attributing them to the brief or marking them complete from a build.

- SRS performance numbers conflict: operations/reports within 10 seconds versus
  patient display 2 seconds, booking 3 seconds and reports 5 seconds. Select a
  repeatable measurement profile and use the stricter relevant threshold until the
  team records another agreed interpretation. The 100-concurrent-user and
  100,000-patient report targets are **not measured**.
- Clinic-hours 99% availability is **not verified**; a health response or deployment
  job cannot prove it. Provider tier/monitoring and a measured observation window
  are operational evidence, not local source changes.
- Authentication/role/ownership/CSRF guards and login audit rows exist. A comprehensive
  before/after business audit and configurable inactivity timeout are not certified
  as implemented. Current token expiry is not an inactivity timer.
- Recovery snapshots were recorded for the earlier authorized shared refresh;
  scheduled backups and a successful restore drill remain operational acceptance,
  not a consequence of having seed SQL.
- English, LKR and Sri Lankan clinic time are supported. Consistent DD/MM/YYYY display,
  browser accessibility and print layout still need screen-by-screen acceptance.

These open targets are not removed to make the checklist look complete. Additional
portal, public-directory, administration, export, hosting and container features
remain available; they do not substitute for primary database/workflow acceptance.

## Release and database application gate

The new `20261012_original_brief_integrity.sql` is forward-only, validates existing
assignment/lifecycle consistency before replacement and performs no data deletion,
backfill or seed replay. It was compiled/exercised only in an isolated local
PostgreSQL container. **It has not been applied to shared Neon.** Both applications
remain on development; hosted main and cloud configuration were not changed.

Before final acceptance, run the cross-branch registration/read → walk-in/booking →
reschedule → notes/multiple treatments → completion → invoice → partial/full receipt →
claim → five filtered reports journey on the intended candidate. Record environment,
repository SHAs, role, expected/actual result and sanitized evidence. Apply the new
migration only after an authorized target/preflight/recovery review; never use the
numbered fresh pipeline as a populated-database upgrade.
