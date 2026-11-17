# ADR-0002: Year-keyed balances with structural yearly reset (no carry-over)

Date: 2026-11-17 · Status: Accepted · Approved in #7 · Research: #8 (`research/data-model.md`)

## Context

The domain fixes: admin-configurable leave types with yearly entitlements, entitlements reset
yearly, no carry-over. A mutable single "balance" row per user/type would need reset logic that
someone must remember to run and un-carry correctly, and config changes could silently rewrite
past entitlements.

## Decision

Model entitlement as a fact keyed by year: **`leave_balances` with `UNIQUE (user_id, leave_type_id,
year)`**, holding a **snapshot** `entitled_days` plus `used_days` and `pending_days`.

- **Yearly reset is structural:** a provisioning job inserts fresh rows for the new year with
  entitlements copied from current `leave_types` config and zeroed counters. No row is mutated;
  unused days physically cannot carry forward; prior years stay queryable.
- **Requests are funded by a composite FK:** `leave_requests (user_id, leave_type_id, balance_year)`
  references `leave_balances (user_id, leave_type_id, year)`, so a request cannot exist without a
  balance row for the year it draws from.
- **Idempotent reset** per (user, type, year) — a rerun must not double entitlement.
- Within-row guard `CHECK (used_days + pending_days <= entitled_days)`; day arithmetic uses
  `DECIMAL(6,2)`; `balance_year` is always the year of `start_date` because requests may not span
  calendar years (see ADR-0006).

## Consequences

- Admin config changes never rewrite history (snapshots).
- The reset is one insert job, not a migration of state.
- Row-local CHECKs cannot express overlap prevention or available-balance checks; those are
  transaction-scoped application rules with `SELECT ... FOR UPDATE` (spec §7.3).

## Alternatives considered

- **Mutable per-user balance with reset logic** — rejected: reset/carry rules become imperative
  code that must be proven, and history is lost.
- **Accrual plans (monthly/hourly earning)** — rejected: out of scope; the domain decision fixes
  yearly entitlements.
