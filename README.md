# Employee Leave Management System

An educational college final project for designing and researching a secure employee leave management system.

This project is **not intended to be production software for companies**. Its purpose is to help the group learn about system design, collaboration, and security features that could be useful in a leave-management application.

## Project overview

The system allows employees to request leave electronically, managers to review and decide requests,
and administrators to manage the system's rules, users, balances, reports, and security trail.

The approved design is a local-development Laravel 13 application using PHP 8.3+, MySQL 8, and
server-rendered Blade views. The implementation is separate from the research and specification work
and will be built from the approved specification.

### Roles

- **Employee** — views balances and personal records, submits and cancels pending requests, tracks
  decisions, uploads required supporting documents, and edits permitted profile fields.
- **Manager** — approves or rejects requests from direct reports, with approval context and mandatory
  rejection reasons. Managers also use the employee leave features for their own requests.
- **Administrator** — manages users, managers, leave types, entitlements, reports, audit logs, and
  exceptional overrides or cancellations. Administrators do not silently replace the normal approval
  workflow.

The professor's reference uses the word "Administrator" for both approval and system-management
responsibilities. This project separates those responsibilities between Manager and Administrator as
a least-privilege design.

### Main capabilities

- Leave requests with leave type, date range, reason, server-computed working days, balance checks,
  overlap checks, and status history.
- Yearly leave entitlements with no carry-over and administrator balance adjustments.
- Manager approval queue with approval or rejection notes and decision context.
- In-app notifications for submissions, decisions, reminders, and administrative actions.
- Secure supporting-document uploads for leave types that require them: PDF/JPEG/PNG, MIME and
  extension validation, 5 MB limit, non-public storage, and policy-controlled downloads.
- Administrator request archive, dashboard counts, leave-balance reports, and leave-transaction
  reports with CSV export and print-ready HTML views.
- Limited employee profile editing for phone and address; identity and reporting fields remain
  administrator-managed.
- Authentication, role-based access control, ownership checks, input validation, CSRF protection,
  SQL-injection prevention, secure sessions, password hashing, and append-only audit logging.

## Scope decisions

The following are intentionally out of scope for version 1:

- Multi-level approval, delegation, and self-approval.
- Leave accrual plans, carry-over, half-days, and hourly leave.
- Payroll, timesheet, calendar integrations, email/SMS/push delivery, and native mobile apps.
- SSO/MFA user interface, impersonation, internationalisation, dark mode, and keyboard shortcuts.
- Admin-managed holiday storage; version 1 uses a fixed application holiday list.

Supporting documents were originally planned as a reference field. The decision was reopened because
the professor's reference explicitly requires uploads and validation. The current decision is recorded
in [ADR-0007](docs/adr/0007-attachment-storage.md).

## Getting started

### Get the repository

1. Open the repository on GitHub.
2. Select **Code** and copy the repository URL.
3. Clone it on your computer:

   ```bash
   git clone https://github.com/Wynteriall/leave-system
   cd leave-system
   ```

4. Open the folder in your editor or development environment.

### Update your local copy

Before starting work, update the local `main` branch:

```bash
git switch main
git pull origin main
```

The application implementation has not yet been added to this repository. When implementation begins,
the expected local environment is:

- PHP 8.3+ with required extensions
- Composer
- MySQL 8
- Node 20+ only if frontend assets are customised
- Optional Laravel Sail for a pinned Docker environment

The approved implementation plan is documented in
[`docs/spec/spec.md`](docs/spec/spec.md), including the expected Laravel setup and seed data.

## How to contribute

Work should normally begin with a GitHub Issue. Do not start a large change without first checking whether an issue already exists.

1. Open the repository's **Issues** tab.
2. Look for an open issue that interests you and check whether it is already assigned or blocked.
3. If no suitable issue exists, create one describing the question, problem, research task, or feature.
4. Ask questions and discuss the issue before making uncertain decisions.
5. Create a branch from `main` for the work:

   ```bash
   git switch main
   git pull origin main
   git switch -c <short-description>
   ```

6. Make focused changes and update relevant documentation.
7. Commit and push the branch:

   ```bash
   git add .
   git commit -m "Describe the change"
   git push -u origin <short-description>
   ```

8. Open a pull request on GitHub, link it to the issue, and request review.
9. Address review comments and merge only after the group agrees the change is ready.

Keep issues, branches, and pull requests focused on one understandable piece of work.

## Research and project knowledge

Research should be written in Markdown and committed to the repository when it becomes useful to the project. Link research to the GitHub Issue that requested it so that others can find the reasoning later.

Before starting new research:

- Search existing GitHub Issues and repository documentation.
- Check whether the question has already been researched.
- Record sources, findings, assumptions, and unresolved questions.
- Explain how the research affects the system or a future decision.

Important project terminology belongs in [`GLOSSARY.md`](GLOSSARY.md). Significant technical
decisions belong in [`docs/adr/`](docs/adr/). The complete approved design is in
[`docs/spec/spec.md`](docs/spec/spec.md), and the professor's reference is retained in
[`docs/employee_leave_application_system.md`](docs/employee_leave_application_system.md).

Relevant documentation:

- [`docs/spec/spec.md`](docs/spec/spec.md) — integrated project specification and acceptance criteria.
- [`research/feature-modules.md`](research/feature-modules.md) — feature research and rationale.
- [`research/security-measures.md`](research/security-measures.md) — security research and demo ideas.
- [`research/data-model.md`](research/data-model.md) — database research.
- [`research/tech-stack.md`](research/tech-stack.md) — technology and architecture research.
- [`docs/adr/`](docs/adr/) — accepted architecture and domain decisions.
- [GitHub issue #11](https://github.com/Wynteriall/leave-system/issues/11) — professor-spec alignment
  decisions.

## Working with AI agents

Group members may use an AI agent such as ChatGPT, Claude, Gemini, or another coding assistant. The person using the agent remains responsible for reviewing its work.

Use a prompt like this:

> You are contributing to the Employee Leave Management System, an educational college project. First read `README.md`, `AGENTS.md`, the relevant GitHub Issue, and related documentation. Explain your plan before making changes. Keep the work limited to the issue, ask questions when requirements are unclear, follow the repository's existing conventions, and update documentation when needed. Do not invent project requirements. Before finishing, check the changes, run relevant tests or checks, and summarize the files changed, decisions made, and verification performed.

An agent should not close an issue, merge a pull request, or make security-sensitive decisions without the contributor reviewing the result.

## Current repository guidance

- GitHub Issues are the source of truth for project tasks and research questions.
- Read [`docs/agents/issue-tracker.md`](docs/agents/issue-tracker.md) for the repository's issue workflow.
- Read [`docs/agents/domain.md`](docs/agents/domain.md) before changing domain terminology or recording technical decisions.
- Treat security as a learning goal: document risks, assumptions, and limitations rather than claiming the system is production-ready.
