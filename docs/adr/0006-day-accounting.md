# ADR-0006: Day accounting — Mon–Fri calculator, holiday constant, whole days, no cross-year requests

Date: 2026-11-17 · Status: Accepted · Approved in #7 (data model grilling), reconciled with #6 ·
Research: #8, #2

## Context

`days_requested` must be computed deterministically before any balance arithmetic is meaningful.
Options ranged from a DB-backed admin-configurable working calendar (`calendar_exceptions` table +
A-06 admin screen, approved as P0 in #6) to a trivial all-days count. The data-model grilling
(#7) chose a middle path; this ADR records it and its interactions with decisions made in #6.

## Decision

1. **Working days are Monday–Friday**, excluding a **fixed holiday list kept as an application
   constant** (not a table). No `calendar_exceptions` table in v1; the working-day calculator is
   one service, one date format, one timezone rule.
2. **Whole days only in v1** — half-days and hourly leave are out of scope (confirmed in #6; the
   schema's `DECIMAL(6,2)` leaves room without implying the feature).
3. **Requests may not span two calendar years.** The server rejects
   `YEAR(start_date) <> YEAR(end_date)` at submit (blocked in v1, per #6; the #7 "require
   splitting" decision names the employee-side path: submit one request per year — cheap with
   E-14 duplication). This also pins `balance_year` = year of `start_date` and keeps overlap
   checks within a single year.
4. **Consequence for #6's feature tiers:** the admin public-holiday-management screen (A-06) is
   **deferred** — it needs holiday storage, which v1 does not have. The holiday list is editable
   in code between demos. This is the one feature-tier adjustment caused by #7.

## Consequences

- `days_requested` is reproducible and demo-stable; the calculator is trivially unit-testable.
- Holiday changes require a code change and redeploy — acceptable for a lab, revisited if A-06 is
  ever promoted back.
- No cross-year edge cases in balance arithmetic, overlap checks, or the reset job.

## Alternatives considered

- **`calendar_exceptions` table + A-06 admin screen** — more realistic and admin-friendly, but
  adds a table, an admin surface, and demo moving parts that #7 declined for v1.
- **Count all calendar days** — simplest, but weekends consume balance and the demo looks wrong.
- **Charge cross-year requests to the start year** — one-line rule, but silently drains one year's
  balance for a Jan 2 return; splitting is cleaner.
