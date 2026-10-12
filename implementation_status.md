# Implementation & Integration Status — MedSync / CATMS

## Checkpoint

Updated **12 October 2026, Asia/Colombo (UTC+05:30)**. This replaces the older October 8 main-branch descriptions.

| Repository | Reviewed branch | Source checkpoint |
| --- | --- | --- |
| CATMS-Backend | `development` | `9e78cefe2b0ef9b851d8308da8699925583daee0` |
| CATMS-Frontend | `development` | `2a941f674ecc6e229d0747eeef2d09c982264108` |

These are local development commits, not a verified hosted release. The 12 October follow-up corrected the SQL artifacts, refreshed the explicitly authorized shared Neon dataset and replaced the landing page's hardcoded clinic data with a public database-backed directory. Both application working trees are clean; no push or application-main merge occurred.

**Implemented** below means the current source contains the workflow. **Acceptance pending** means actual role-scoped browser/API/PostgreSQL results are still required. Neither static checks nor a successful deployment job establish complete functionality.

## Current source status

| Area | Implemented in development | Remaining acceptance / limitation |
| --- | --- | --- |
| Authentication | Username or account-email login; cookie session confirmation; current authority rechecks; shared write CSRF and session-reset safeguards; threadpool password operations. | Cross-origin sessions, expired/changed authority and legacy ambiguous accounts need runtime checks. |
| Patient registration/portal | Staff registration with optional linked portal account/contact in one transaction; cross-identifier collision checks; own-record pages and nested contact operations. | Optional provisioning/change-password journey, rollback/collision cases and patient editable-field policy. Initial password rotation is advised, not enforced. |
| Scheduling | Normal/walk-in creation, cancelled-slot handling, qualified list filters/status/display fields, cancel/complete/reschedule and retained history. | Competing requests, role/branch denial, date/time round trips and all status transitions on PostgreSQL. Scheduling is staff-managed. |
| Clinical records | Multiple captured-price treatments, consultation notes, cancelled/invoiced edit guards and doctor-own scope. | Note authoring/retention policy, care-to-completion journey and denied clinical access. |
| Billing | Completed-visit invoice generation; one invoice/visit; issued-item write lockout; derived totals; separate patient receipts and doctor compensation. | Partial payment/overpayment/rollback, totals and compensation reconciliation; no dedicated refund/adjustment UI claimed. |
| Payment UX | Confirmed receipt versus failed refresh versus unknown write outcome; duplicate-submit prevention. | Real interruption/reconciliation scenarios. No general automatic write retry or exactly-once delivery guarantee. |
| Insurance | Policy dates, grouped treatment coverage, pending/approved reservations, invoice-wide ceiling, partial approval, doctor-own claim reads and coverage estimates. | Matching SQL rule installed and all existing claims revalidated; concurrent-session and legacy rejection journey acceptance remain open. No insurer settlement or annual-benefit model. |
| Reports | Five matching API/UI contracts; period/branch scopes; correct financial labels; loaded filter snapshots; native Print / Save as PDF. | Known-result comparison, allow/deny checks and browser/PDF visual QA. No server-generated PDF. |
| Administration | Treatment/category management; Staff/Branches load/save guards; manager billing controls aligned with API. | Privileged-account, branch-assignment and failed-load/save journeys. |
| Public directory | Database-backed branch names/locations/contacts, specialties and doctor/service counts through one public brochure-only endpoint; no invented hours or hardcoded data fallback. | Rebuild/run both development applications; verify load, failure/retry and admin updates in a browser. Older hosted backend builds do not contain this new endpoint. |
| Responsiveness and cleanup | In-flight read coalescing/invalidation, selected stale-response guards, in-page dialogs and removal of unused prototype forms/starter icons. | Browser navigation/accessibility and measured response/load targets; no benchmark claim. |
| Runtime safety | Production configuration rejects default/short/equal secrets and insecure cookies; pool bounds validated. | Requires explicit production environment settings; actual cloud settings were not inspected. |

## Database and release gates

1. **Insurance validation:** the matching claim function/trigger was PostgreSQL-validated and installed on the authorized shared target on October 12. Numbered functions/triggers are aligned. Status-only revalidation passed for all existing claims. Concurrent competing claim/coverage writes and real role journeys are not yet certified.
2. **Schema/data readiness:** October 12 maintenance consolidated to three branches, six insurers and six service groups, removed generated QA labels and corrected cross-branch links. Linked identities/history were retained; captured financial amounts and passwords were verified unchanged. Source `09_seed_data.sql` is for an empty schema, not replay on shared Neon.
3. **Hosted version:** the team reports main is hosted and development is not hosted. Current deployed SHAs and cloud configuration were not verified. The hosted demonstration may therefore differ from this candidate.
4. **Release control:** continue both applications on exact `development`. Do not merge the candidate to `main` without explicit release-owner authorization; a documentation update does not authorize deployment or database changes.
5. **Operational review:** confirm patient field-level editing and clinical-note policy, legacy account/role resolution, audit coverage, backup/recovery and session-expiry behavior. Current guards and login records are not a comprehensive security or retention certification.

## Checks already recorded

Current follow-up checks:

- Isolated PostgreSQL fresh-schema setup and complete seed passed twice, with all changes rolled back; all 22 tables received linked data.
- The in-place refresh passed rollback validation, then committed with encrypted recovery snapshots. Post-commit independent read-only SQL assertions and the actual public-directory query/response contract passed.
- Financial preservation checks covered captured treatment/invoice prices, patient receipt amounts, doctor compensation and filed claim amounts/statuses. Password preservation and branch/history/identity consistency checks passed.
- Backend AST passed 56 files; dummy-environment OpenAPI generation passed 130 operations / 75 paths.
- Frontend TypeScript no-emit, in-memory Vite production build (12 outputs) and both diff checks passed.
- Sandboxed in-process HTTP checks stalled and were stopped. A local event-loop diagnostic and an outside-sandbox ASGI HTTP projection check then passed with dummy configuration and a fake database; no server listener or shared mutation was involved. No browser, print-layout, load-target or concurrent-session acceptance is claimed.

Earlier isolated handler/component checks remain dated evidence, not a replacement for the remaining full-stack journeys. The new directory API exposes only brochure data and aggregates; admin-only branch management permissions were not widened.

## Team acceptance handoff

Use approved synthetic records and record repository SHA, environment, role, steps, expected/actual result and sanitized evidence. Assign a teammate and reviewer when a case is taken; no ownership is inferred from this checklist.

| Priority | Journey | Completion evidence |
| --- | --- | --- |
| Release prerequisite | Validate the insurance migration and lock ordering on isolated PostgreSQL. | Compilation, shared application and existing-claim status revalidation are recorded; finish legacy rejection and competing claim/coverage checks. |
| Core workflow | Staff registers optional portal → username/email login → password change → staff booking → care → completion → invoice → partial receipt. | Persisted links, captured prices, correct states/totals and no duplicate accounts/invoices/receipts. |
| Lifecycle | Normal/walk-in booking, overlap denial, cancel/reschedule, retained history and date/time display. | Refresh retains the expected state; concurrent non-cancelled slots cannot overlap. |
| Authorization | All five roles, own/other patient, own/other doctor and own/other branch. | Allowed paths succeed; denied IDs/roles return appropriate errors with no leaked data or writes. |
| Finance and insurance | Pending reservation, partial approval, rejection, multiple policies, uncovered treatment and unknown-payment outcome. | Request/approval/receipt meanings reconcile independently; no accidental second receipt. |
| Reports | All five reports, matching date/branch filters, empty/failure states and printed/PDF long tables. | Screen/export/known totals agree; report scope cannot be widened. |
| Administration and UX | Catalogue, Staff/Branches, contacts/profile, failed loads, duplicate clicks, route refresh and navigation. | Clear in-page states, no stale overwrite, correct fields/permissions and usable keyboard/layout behavior. |
| Delivery | Reconcile ER/SRS, schema migration record, source SHAs and authorized release configuration. | Reviewed release evidence, then separately authorized main integration/deployment. |

## Published starting-baseline references

The links below pin the previously published baseline. The current local development commits above were not pushed in this maintenance pass.

- [Backend endpoints at reviewed SHA](https://github.com/datanexus-cs3043/CATMS-Backend/tree/4f22ab42a011072a4d276359dcb2c4d54f139b41/app/api/v1/endpoints)
- [Backend schemas](https://github.com/datanexus-cs3043/CATMS-Backend/tree/4f22ab42a011072a4d276359dcb2c4d54f139b41/app/schemas)
- [Database artifacts and migrations](https://github.com/datanexus-cs3043/CATMS-Backend/tree/4f22ab42a011072a4d276359dcb2c4d54f139b41/database)
- [Frontend API service](https://github.com/datanexus-cs3043/CATMS-Frontend/blob/6f2333db1cafff0d1c18352e9d35f82f646323c3/src/services/api.ts)
- [Frontend authentication](https://github.com/datanexus-cs3043/CATMS-Frontend/tree/6f2333db1cafff0d1c18352e9d35f82f646323c3/src/auth)
- [Reports page](https://github.com/datanexus-cs3043/CATMS-Frontend/blob/6f2333db1cafff0d1c18352e9d35f82f646323c3/src/pages/Reports.tsx)
