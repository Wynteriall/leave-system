# ADR-0005: In-app-only notifications (email out of scope)

Date: 2026-11-17 · Status: Accepted · Approved in #6, #7 · Research: #2, #10

## Context

Deployment is local-dev only: there is no reliable mail transport in the lab, and an SMTP
dependency is the most likely cause of a failed demo (`research/tech-stack.md`, risk 4). The
feature research still requires that the right person is told at the right time (N-01–N-08).

## Decision

Notifications are **in-app only**: a `notifications` table (recipient, optional request link,
type, `read_at`) rendered as an inbox with an unread badge. Reminders are scheduled commands
(`Schedule::command(...)->dailyAt('08:00')`, run with `php artisan schedule:work`), idempotent per
(request, recipient, rule). The `channel` column exists (`in_app` / `email`) so email is a
*configured channel we did not enable*, not a lie — but no email code ships.

## Consequences

- The workflow works entirely inside the app; notification creation sits in the service layer next
  to the state change, inside the same transaction, so a decision can never exist without its
  notification.
- Notification bodies are field-limited: they never carry more than the recipient may already see
  (no reasons to colleagues, no personal data) — part of the privacy acceptance criterion.
- Demo hygiene: seed data includes a recent submission so the queue/notification demo is live, and
  idempotent reminders prevent a wall of stale notifications.

## Alternatives considered

- **Email delivery via SMTP/log driver** — rejected: transport unreliability in the lab; deferred
  without modelling changes.
- **Push/SMS** — rejected: external services, trust boundaries, zero demo value locally.
