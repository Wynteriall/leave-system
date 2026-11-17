# Glossary

Domain vocabulary for the employee leave management system. Use these terms exactly as defined
here; do not drift to synonyms. Decisions live in `docs/adr/`; the full specification in
`docs/spec/spec.md`.

## People and roles

- **Employee** — any user of the system. Employees, Managers and Administrators are all users;
  "employee" without qualification means a user acting for themselves (requesting, tracking,
  cancelling their own leave).
- **Manager** — a user with approval authority over their **direct reports**. A manager is also an
  employee (has their own balances and requests).
- **Administrator** — a user who configures the system (users, roles, leave types, balance
  adjustments) and can **override** or **cancel** any request. Exactly three roles exist, seeded
  and not user-editable at runtime.
- **Role** — one of `employee`, `manager`, `administrator`; stored as an FK (`role_id`) on the
  user, never trusted from the client.
- **Reporting line / manager link** — the self-referencing `users.manager_id`. It routes
  approvals: a request goes to the submitter's manager at submission time. A user has at most one
  manager; a manager may have many reports; top-level users have none.
- **Direct approver** — the manager who received a request via the reporting line. The only
  non-admin role allowed to see a request's free-text **reason**.
- **Last-admin guard** — the rule that the last Administrator cannot be demoted or deactivated.

## Leave configuration

- **Leave type** — an admin-configured category of absence (e.g. `ANNUAL`, `SICK`, `UNPAID`) with
  a yearly entitlement, a `requires_document` flag, and an `is_active` flag. Deactivated, never
  hard-deleted, so history stays readable.
- **Entitlement** — the number of days a leave type grants per year. The **current configured**
  value lives on the leave type; each balance **snapshots** it, so later config changes never
  rewrite past years.
- **Yearly reset** — the January 1 provisioning of fresh balance rows for the new year. Structural,
  not a mutation: prior-year rows are kept queryable.
- **Carry-over** — none, by design: unused days are deliberately not copied into the new year's
  balances. "No carry-over" is a property of the schema, not a remembered rule.
- **Holiday list** — the fixed set of non-working public holidays, an application constant in v1.
  The admin holiday-management screen is deferred (ADR-0006).

## Day accounting

- **Working day** — a Monday–Friday date not on the holiday list. The only unit the system counts.
- **Calendar day** — a plain date; used in request date ranges, never for arithmetic.
- **Working-day calculator** — the single server-side service that computes `days_requested` from
  a date range. One service, one date format, one timezone rule.
- **Whole days only** — v1 policy: no half-days or hourly leave (out of scope).
- **`days_requested`** — the server-computed number of working days in a request. Never taken from
  a client-supplied value.

## Balances

- **Leave balance** — one row per **user × leave type × year**: `entitled_days` (snapshot),
  `used_days`, `pending_days`. The composite key `(user_id, leave_type_id, year)` is the heart of
  the entity.
- **Used days** — days consumed by **approved** requests.
- **Pending days** — days reserved by **pending** requests. Released on rejection/cancellation.
- **Remaining** — `entitled − used − pending`; derived, never stored. Displayed wherever a
  decision happens (HCI acceptance criterion).
- **Balance adjustment** — an admin correction to one person's balance for one type and year,
  with a mandatory reason, recorded in the audit log. Admin adjustments may create a negative
  remaining balance; employee requests may not.

## Requests and decisions

- **Leave request** — the central transaction: type, date range (one calendar year only),
  `days_requested`, reason, status, decider. Funded by a composite FK to the exact balance row.
- **Request status vocabulary** — closed: `pending`, `approved`, `rejected`, `cancelled`.
- **Decision** — a manager's approve/reject of a pending request. Recorded as a row in the
  append-only **approval timeline**.
- **Override** — an Administrator replacing the current decision (e.g. overriding a rejection).
  Appends a new approval row with `is_override = TRUE`; the original decision is never rewritten.
- **Cancellation** — removing a request's effect on the balance. Pending requests: the employee
  cancels their own. **Approved** leave: admin-mediated only (E-12 raises a withdrawal; A-07
  records it and restores the balance).
- **Self-approval** — impossible. A manager's own request routes to their manager, or to an
  Administrator if they have none.
- **Approval timeline (`approvals`)** — append-only per-request decision history: submitted,
  approved, rejected, overridden, cancelled, with actor, timestamp, note, `is_override`.
- **Overlap** — two requests of the same user whose date ranges intersect. Blocked at submit for
  any pair of `pending`/`approved` requests.
- **Cross-year request** — a request spanning 31 Dec → Jan. Blocked by server validation; the
  employee submits one request per year instead.

## System trail

- **Audit log (`audit_log`)** — append-only trail of security and configuration events (logins,
  role changes, leave-type changes, balance adjustments, overrides, cancellations) with actor,
  polymorphic subject, JSON metadata, IP. Distinct from the approval timeline: domain decisions
  vs. the broad security trail.
- **Notification** — an in-app inbox row for one recipient about one event, optionally linked to a
  request. Never carries more than the recipient may already see.
- **Reminder idempotency** — at most one open reminder per (request, recipient, rule); a scheduled
  job never stacks copies.
- **Nudge** — an employee-initiated reminder to their approver, max once per 24 h per request.
- **Soft deactivation** — users and leave types get `is_active` instead of deletion, preserving
  FK targets in history.
- **Attachment** — a supporting document associated with a leave request when its leave type
  requires one. In v1, one PDF/JPEG/PNG file up to 5 MB is stored on a non-public disk and is
  downloadable only by the employee owner, direct approver, or Administrator.
- **Permitted profile fields** — the employee-editable `phone` and `address` fields. Identity and
  routing fields (`name`, `email`, `employee_code`, `role`, `manager`) remain Administrator-managed.

## Security terms

- **Data minimisation** — every screen, export, notification and log entry shows only the fields
  its recipient's job requires (approved acceptance criterion in #6).
- **Insider threat** — the primary threat-model lens (approved in #9): employees, not outside
  hackers, are the greatest risk (misuse, malpractice, greed).
- **Append-only** — written once, never updated or deleted; enforced by convention and narrow DB
  grants.
