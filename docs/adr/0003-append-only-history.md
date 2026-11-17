# ADR-0003: Append-only decision and audit history

Date: 2026-11-17 · Status: Accepted · Approved in #7 · Research: #8 (`research/data-model.md`)

## Context

The domain fixes a single-approval workflow where the Administrator can override or cancel.
"Destructive override" (rewriting the manager's decision) would make abuse invisible and the
security story unpresentable. MySQL has no declarative immutable-table construct.

## Decision

Two append-only tables, deliberately separate:

- **`approvals`** — the per-request decision timeline (`submitted`, `approved`, `rejected`,
  `overridden`, `cancelled`) with actor, `to_status`, note, `is_override`. An admin override or
  cancel **adds a row** and updates the request's current status; the original decision row is
  never updated or deleted.
- **`audit_log`** — the broad security/config trail (auth events, role changes, leave-type and
  balance changes, overrides, cancellations) with actor, polymorphic subject, JSON metadata, IP.

Enforcement is convention + defence in depth: application code never issues `UPDATE`/`DELETE`
against these tables, and narrow DB grants back that up.

## Consequences

- Every exception is explainable after the fact — the admin audit view (A-08) is the most
  persuasive screen in the security presentation (OWASP A09).
- The admin's elevated access is a logged event, not an implementation accident (privacy criterion).
- Storage grows monotonically; irrelevant at lab scale. No retention/purge policy is modelled
  (local dev, synthetic data).

## Alternatives considered

- **Mutable request columns as the only history** — rejected: loses who-did-what-when.
- **Single combined event table** — rejected: mixes domain decisions with security telemetry;
  different access policies and shapes.
