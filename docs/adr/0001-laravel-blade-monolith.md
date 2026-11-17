# ADR-0001: Laravel 13 (PHP 8.3+) + MySQL 8, server-rendered Blade monolith

Date: 2026-11-17 · Status: Accepted · Approved in #3 · Research: #10 (`research/tech-stack.md`)

## Context

The system is a secure employee leave management system for a security-course lab: it must be
learned, built, defended, and presented quickly (research landed the night before the Nov 18
presentation). Team background: Java basics, MySQL, PHP/Laravel; React used in a separate program;
plain HTML/JS excluded; a "middle-difficulty" stack was requested. Deployment is local-dev only.

## Decision

Build a single server-rendered monolith: **Laravel 13 on PHP 8.3+ with MySQL 8**, Blade templates,
session-cookie auth with CSRF tokens, Fortify auth scaffolding, Eloquent ORM with transactional
domain services. **No separate React SPA.** Laravel Sail is an optional Docker path for Windows
toolchain friction.

Security controls ride framework defaults: Eloquent/PDO parameter binding, `ValidateCsrfToken`
middleware on the `web` group, Blade auto-escaping, `Hash` facade (Argon2id primary — one
`HASH_DRIVER` env line; bcrypt as documented fallback), Fortify login rate limiting, role
middleware + record-level policies.

## Consequences

- One codebase, one language, one start command; validation lives once, server-side, in Form
  Requests; no CORS, no token storage decisions.
- Full-page interactions only (plus minimal vanilla JS); no client-side routing — accepted, since
  no feature requires it.
- Every team machine needs PHP 8.3+, Composer, MySQL 8 (or Docker for Sail).
- The "framework did the security" objection is answered per control by showing the enabling
  configuration plus one failing request that proves it active.

## Alternatives considered

- **Django + MySQL** — strong defaults, but Python is outside the team's studied set, and Django
  admin would bypass the custom role/authorization story.
- **Spring Boot + MySQL** — reference-quality security DSL, but Spring ≠ "Java basics"; lab time
  would go to wiring, not the domain.
- **ASP.NET Core + MySQL** — zero team familiarity; defaults assume SQL Server.
- **Laravel + Inertia + React** — sanctioned escalation path *if* a React artefact is separately
  mandated; deliberately not the default.
