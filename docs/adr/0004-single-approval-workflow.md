# ADR-0004: Single-approval workflow with admin override/cancel

Date: 2026-11-17 · Status: Accepted (domain decision; recorded in #6, #7) · Research: #2, #8

## Context

The domain decisions were fixed before research began: exactly three roles; a manager
approves/rejects; the Administrator can override/cancel. Research confirmed that multi-level
approval chains would double the state machine for zero demo value (Odoo ships four approval
modes precisely because it is a general HR suite).

## Decision

- **One decision per request:** the **direct approver** (the submitter's manager via the reporting
  line) approves or rejects. No delegation, no approve-on-behalf, no multi-level chains.
- **Administrator as escape hatch:** may override or cancel any request with a mandatory reason,
  appending to the approval timeline (ADR-0003) — never erasing the original decision.
- **Routing is frozen at submission:** the request stays with the manager who was the submitter's
  manager at submit time, even if the reporting line changes mid-flight; only an admin can
  reassign.
- **Self-approval is impossible:** a manager's own request routes to their manager, or to an
  Administrator if they have none.
- **Cancellation semantics:** employees cancel their own *pending* requests; withdrawal of
  *approved* leave is admin-mediated (E-12 raises it; A-07 records it and restores the balance).

## Consequences

- The state machine stays small: `pending → approved | rejected`, plus admin paths to `cancelled`.
- Approval latency is the system's main failure mode — mitigated by notifications/reminders
  (N-01–N-06) and, ultimately, the admin override.
- Managers who are also employees have two separated areas: "My leave" and "Approvals" (M-11).

## Alternatives considered

- **Multi-level approval / delegation** — rejected: different product, doubles the state machine,
  out of scope.
- **Employee-initiated cancellation of approved leave** — rejected: would let an employee move
  their own balance after the fact; admin-mediated instead.
