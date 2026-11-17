# Feature modules and role use cases

Research ticket: [#2](https://github.com/Wynteriall/leave-system/issues/2)
Map: [#1](https://github.com/Wynteriall/leave-system/issues/1)
Feeds: grilling ticket [#6](https://github.com/Wynteriall/leave-system/issues/6) (approve feature set)
Status: complete, ready for the grilling ticket.
Scope note: feature and usability research only. Security controls are ticket [#5](https://github.com/Wynteriall/leave-system/issues/5); the data model is ticket [#8](https://github.com/Wynteriall/leave-system/issues/8). Where a feature is also a security control, this document says so in one line and leaves the control detail to those tickets.

---

## 1. Summary and recommendation

The system needs **three thin modules plus one shared spine**:

| Module | Who uses it | Job to be done |
| --- | --- | --- |
| **My leave** | everyone (Employee, Manager, Administrator) | see balances, request leave, track and cancel own requests |
| **Approvals** | Manager (+ Admin as override) | decide pending requests with enough context to decide well |
| **Administration** | Administrator | users, leave types, entitlements, holidays, audit log |
| **Notifications** (cross-cutting) | everyone | tell the right person at the right time, and chase the person who is blocking the flow |

Ranking, in one sentence: **the workflow is core, the context around the decision is core, and everything that saves typing is QoL.** A manager who cannot see the employee's remaining balance or who else is out cannot make a good decision, so those are core even though they look like "extras". The reverse also holds: dark mode, bulk actions and CSV import are all nice, and none of them are worth a late night before the presentation.

Counts: **22 core features, 18 QoL features, 8 notification rules.** Recommended cut line for the lab: ship **all 22 core**, ship QoL only where it is a by-product of work already done (for example the status timeline, which is free once approvals are stored).

Three design decisions carry most of the usability value:

1. **The balance is always visible where the decision happens.** Employee dashboard, request form, and manager approval detail all show the same "entitled / used / pending / remaining" numbers. This single rule removes the three most common conversations in a leave system ("how many days do I have", "can I take these dates", "does he have enough left").
2. **Validation happens before submission and again on the server.** Overlap, insufficient balance, weekends and public holidays are reported inline in the form and re-checked server-side. This is usability and business-logic security in the same feature (ASVS 5.0.0 V2.2.2, V2.3.1).
3. **Every decision is explainable after the fact.** A request detail page shows the full timeline (submitted, approved/rejected, overridden, cancelled) with actor, timestamp and note; the admin gets the same data as a filterable audit log. This is the feature that makes the security story presentable, because it is visible on screen rather than argued in a slide.

Deliberate exclusions (section 8): multi-level approval, delegation, accrual plans, carry-over, payroll integration, email delivery, mobile app, impersonation.

---

## 2. How this was researched

The question is a product-design question, not a standards question, so the method was:

1. Read how established leave systems actually model the employee/manager/admin split, from first-party vendor documentation: Odoo Time Off 18 (types, allocations, approval, overview calendar, reporting), Sage HR (approver reminders), SeamlessHR (employee-initiated "nudge"), Nintex (reminder before leave start date).
2. Map every candidate feature onto the already-decided domain rules (three roles, single approval, admin-configurable types, yearly reset with no carry-over) and drop anything that only exists to serve multi-level approval or accrual.
3. Check each feature that is also a control against OWASP ASVS 5.0.0 (authorization, business logic, session, logging) and each UI-facing feature against WCAG 2.2, so "usability" claims are not just opinion.
4. Rank by three questions, in order: does the role's end-to-end job fail without it? does it change what someone can decide? how much does it cost to build?

Assumption stated once: mechanics reference the recommended stack in `research/tech-stack.md` (Laravel monolith, Blade, MySQL, `php artisan schedule:work`). The feature list itself is stack-agnostic; only section 4.5.4 depends on that recommendation.

---

## 3. Role use cases

### 3.1 Employee

**Primary goal:** get time off approved without having to ask anyone what the rules are.

| # | Use case | Success looks like |
| --- | --- | --- |
| U-E1 | Check how much leave is left this year | One screen, per type, with pending requests already deducted |
| U-E2 | Request leave for a date range | Form computes working days, warns about clashes and shortage before submit |
| U-E3 | Find out what happened to a request | Status, decision, decider and note on one page, plus a notification |
| U-E4 | Change plans and withdraw | Cancel pending request in one click; balance is released |
| U-E5 | Plan around the team | See who else is out in the target week, without seeing why |
| U-E6 | Recover access | Password change/reset without an admin (security ticket owns the flow) |

Anti-goals (things the employee must never have to do): email a manager to ask for approval, ask an admin to correct a balance, guess whether weekends count, discover a rejection days later.

### 3.2 Manager

**Primary goal:** clear the approval queue in a couple of minutes, with enough context to defend each decision.

| # | Use case | Success looks like |
| --- | --- | --- |
| U-M1 | See what is waiting for me | Queue defaults to pending requests of direct reports, oldest first, with a count badge |
| U-M2 | Decide | Approve (optional note) or reject (mandatory reason) in one action |
| U-M3 | Decide well | Detail view shows requested dates, computed days, balance before/after, and overlapping team absences |
| U-M4 | Not miss anything | In-app notification on submission, reminder if a request sits unactioned |
| U-M5 | Answer "when did I approve that" | Team request archive filtered by employee, type, status, date |
| U-M6 | Book my own leave | Same "My leave" module as employees; self-approval is not possible |

Anti-goals: approving blind, hunting through email, remembering an entitlement number, being the bottleneck nobody can see.

### 3.3 Administrator

**Primary goal:** keep the rules correct and be able to explain every exception.

| # | Use case | Success looks like |
| --- | --- | --- |
| U-A1 | Onboard a person | Create user, set role and manager; the manager field is what routes approvals |
| U-A2 | Change the policy | Edit leave types and yearly entitlements without breaking history |
| U-A3 | Correct an entitlement | Adjust one person's balance for one type and year, with a reason recorded |
| U-A4 | Roll the year over | Entitlements reset on Jan 1 with an audit entry and a notification |
| U-A5 | Set the working calendar | Public holidays, so "5 days" means 5 working days |
| U-A6 | Handle the exception | Override or cancel any request, with a mandatory reason, without erasing the original decision |
| U-A7 | Prove what happened | Filterable audit log covering logins, role changes, policy changes, overrides, cancellations |
| U-A8 | Answer questions about usage | Utilisation by type, outstanding balances, who is out today |

Anti-goals: editing the database by hand, deleting rows to fix a mistake, being unable to tell who changed what.

---

## 4. Feature catalogue

Legend: **Core** = the role's job cannot be completed without it, or its absence creates a wrong/unsafe outcome. **QoL** = removes friction, does not change what is possible. Priority tags (P0/P1/P2) are consolidated in section 5.

### 4.1 Employee module (E)

| ID | Feature | Class | Rationale |
| --- | --- | --- | --- |
| E-01 | Sign in / sign out, landing page chosen by role | Core | Entry point for every use case; the role gate here is the base of function-level authorization (ASVS V8.2.1). |
| E-02 | Balance dashboard per leave type: entitled, used, pending, remaining, current year | Core | Answers the most frequent question in one screen and prevents requests that are doomed on arrival (Odoo "My Time" / balance reporting). |
| E-03 | Request form: type, start date, end date, reason, server-computed working days | Core | The product's reason to exist; computing days server-side is also the business-logic control that blocks tampered requests (ASVS V2.3.1). |
| E-04 | Pre-submit checks with inline messages: overlap with own requests, insufficient balance, weekend/holiday-only range, past dates, inactive type | Core | Cheapest usability win in the system: the employee fixes the problem, the manager never sees an invalid request (ASVS V2.2.2 requires the server check anyway). |
| E-05 | My requests list with status, year/status/type filters, and detail page | Core | Status transparency is what stops "did you get my request?" messages; filters are one query. |
| E-06 | Cancel own pending request | Core | Frees the reserved balance immediately and removes work from the admin, with no new permission model. |
| E-07 | In-app notification inbox with unread badge and mark-as-read | Core | Decision latency is this system's main failure mode; in-app is the only delivery channel that works in a local-only deployment. |
| E-08 | Account self-service: view profile and manager, change password | Core | Required by ASVS V6.2.2/V6.2.3 and it is the honest self-service story to demo instead of admin-set passwords. |
| E-09 | Request timeline: submitted, decided, overridden, cancelled, with actor, time and note | QoL | Answers "why" without a support conversation; nearly free once approvals are append-only. |
| E-10 | My year calendar: own leave plus public holidays | QoL | Planning aid; reuses the calendar component needed by E-11 and M-08. |
| E-11 | Team availability calendar (name + leave type only, no reason) | QoL | Prevents clashes and is the most demo-visible usability feature; field-limited for privacy (ASVS V8.1.2). |
| E-12 | Withdraw an already-approved request (routes to manager/admin; balance restored on approval of the withdrawal) | QoL | Plans change; without it people ask an admin to edit data, which is exactly the habit a secure system should not teach. |
| E-13 | Year-end reminder listing unused days that will expire | QoL | With no carry-over, unused days are a real loss; one scheduled query. |
| E-14 | Duplicate an old request ("request again") | QoL | Repeat patterns (monthly day off) are common; saves form-filling, adds no new rule. |
| E-15 | Export own approved leave as .ics | QoL | Personal calendar value with no OAuth, no external API, no new trust boundary. |
| E-16 | Supporting-document upload for types flagged as requiring one | QoL | Only needed if the admin sets the flag; the cheap version is a document-reference field, because a real upload pulls in the whole file-handling surface (ASVS V5). Decide in the grilling ticket. |
| E-17 | Draft autosave for a half-filled request | QoL | Five fields do not justify the state machine; cut first if time is short. |

### 4.2 Manager module (M)

| ID | Feature | Class | Rationale |
| --- | --- | --- | --- |
| M-01 | Approval queue defaulting to "waiting for me" (direct reports), oldest first, with a count badge | Core | The manager's single job; Odoo ships this exact default filter ("Waiting For Me") because an unfiltered list buries the work. |
| M-02 | Approve with optional note, one action from list or detail | Core | Every extra click here is a day of employee waiting; approval must never require a separate edit form. |
| M-03 | Reject with mandatory reason | Core | Fairness and accountability; a rejection without a reason is indistinguishable from a mistake, and the reason is required metadata for the log (ASVS V16.2.1). |
| M-04 | Approval detail with decision context: computed days, balance before/after, employee's other pending requests, overlapping team absences | Core | Approving without balance and coverage context is a rubber stamp; this screen is where the domain decision is actually made. |
| M-05 | Notified on submission (badge + inbox), plus reminder for stale requests | Core | Push and pull: a queue nobody opens is not a workflow; reminder cadence in section 4.5. |
| M-06 | Team request archive: all statuses, filter by employee/type/date/status | Core | The queue is a dead end after a decision; this is the same query with a different filter and answers "when did I approve that". |
| M-07 | Team balances overview (remaining per person per type) | QoL | Useful for spotting exhausted or unused entitlements before the year ends; the admin report covers the formal need. |
| M-08 | Team coverage calendar, including pending requests (solid vs striped) | QoL | Real capacity planning; overlaps with E-11 and M-04, so build it once and reuse it. |
| M-09 | Bulk approve selected requests | QoL | Saves clicks on a Monday morning but encourages rubber-stamping; if built, require a confirmation that names the count. |
| M-10 | Ask a clarifying question without deciding (comment thread on a pending request) | QoL | Avoids reject-and-resubmit loops, but adds a state and a thread to the workflow; only worth it if rejection feels too blunt in the demo. |
| M-11 | Clear nav separation between "My leave" and "Approvals" | QoL | Managers are also employees; without visible separation they cannot find their own leave and the queue at the same time. |
| M-12 | Export team leave as CSV | QoL | Rarely needed by a manager at lab scale; the admin report satisfies the requirement once. |

### 4.3 Administrator module (A)

| ID | Feature | Class | Rationale |
| --- | --- | --- | --- |
| A-01 | User management: list/search, create, edit, deactivate, assign role, assign manager | Core | The `manager_id` reporting line is what routes approvals, so nothing else works until this exists (ASVS V8.2.1). |
| A-02 | Guardrails on identity changes: cannot remove/demote the last admin, deactivation ends active sessions, no self-approval bypass | Core | Three cheap rules that prevent the two classic demo disasters (locking yourself out, approving your own request) and satisfy ASVS V7.4.2. |
| A-03 | Leave type management: name, yearly entitlement days, requires-document flag, active flag, never hard delete | Core | The domain decision made this admin-configurable; soft-deactivate keeps historical requests readable. |
| A-04 | Per-user balance adjustment for a type and year, with mandatory reason | Core | Joiners, leavers and corrections are inevitable; without this the only fix is direct database access, which is the opposite of defendable. |
| A-05 | Yearly reset: entitlements reset on Jan 1, no carry-over, audit entry and notification | Core | The domain rule; one scheduled job (`->yearlyOn(1, 1)`) plus an idempotency guard so it cannot run twice. |
| A-06 | Public holiday calendar management | Core | Working-day arithmetic is wrong without it, and wrong arithmetic means wrong balances (Odoo makes the same point about configuring holidays before use). |
| A-07 | Override or cancel any request with a mandatory reason, preserving the original decision | Core | The domain's escape hatch; append a new approval/audit row instead of mutating the old one (ASVS V2.3.5 pattern of multi-party scrutiny on sensitive flows). |
| A-08 | Audit log viewer with filters (actor, event type, subject, date range) | Core | OWASP Top 10 A09 and ASVS V16.3.2 make this a control, and it is the most persuasive screen in a security presentation. |
| A-09 | Admin dashboard: pending count, absent today, overdue queue, utilisation by type | QoL | Situational awareness and a strong opening slide; every number is a query over data that already exists. |
| A-10 | Reports with CSV export (usage by type, balances by person) | QoL | Expected of an HR tool (Odoo's reporting set) but not needed to complete any role's job; keep escaping RFC 4180-correct if built (ASVS V1.2.10). |
| A-11 | Settings: reminder schedule (days, time), session timeout, password policy knobs | QoL | Genuine convenience, but these are configuration surfaces over controls owned by the security ticket, not new logic. |
| A-12 | Session administration: terminate a user's sessions, trigger a password reset | QoL | The controls are ASVS V7.4.5/V6.4.6; the QoL part is doing it from a screen instead of a console. |
| A-13 | Bulk manager reassignment when a manager leaves | QoL | One-off pain; a seeder or script is acceptable at lab scale. |
| A-14 | User import from CSV | QoL | Saves ten minutes once and adds a parsing surface plus formula-injection concerns; defer. |
| A-15 | Org overview: who reports to whom | QoL | Makes A-01 comprehensible at a glance when the seed data has three managers. |

### 4.4 Cross-cutting usability (X)

| ID | Feature | Class | Rationale |
| --- | --- | --- | --- |
| X-01 | Responsive layout, usable at 375 px wide | QoL | Approvals are a phone activity in real life; at lab scale it is a "looks finished" signal rather than a requirement. |
| X-02 | WCAG 2.2 AA basics: real labels, visible focus, 4.5:1 contrast, errors described in text, status messages announced | QoL | SC 4.1.3 exists precisely for notification and flash-message UX, so the accessible version is the same work as the good version. |
| X-03 | One date format, one timezone rule, working-day math in one service | QoL | Off-by-one date bugs are the classic defect in leave systems and they look like policy errors to the audience. |
| X-04 | Search, filter and sort on every list | QoL | Three lists share one Blade component; without it the admin audit log is unreadable at 50 rows. |
| X-05 | Empty states and inline help ("weekends and public holidays are not counted", "unused days do not carry over") | QoL | Pre-empts the questions the presenter would otherwise answer verbally. |
| X-06 | Confirmation dialogs on destructive or irreversible actions | QoL | Cheap protection against a mis-click during a live demo. |
| X-07 | Flash message after every action, with `role="status"` | QoL | Users need to know the click worked; screen readers need it programmatically (WCAG 4.1.3). |
| X-08 | Dark mode | QoL (cut) | Pure cosmetics, zero workflow value; do not build before the presentation. |
| X-09 | Keyboard shortcuts for the approval queue | QoL (cut) | Nice for a power user, invisible in a demo, and it competes with form accessibility work. |

### 4.5 Notifications and reminders (N)

This is the part that makes the workflow feel alive, and the part most likely to be skipped. It should not be: an approval queue with no notification is a to-do list nobody knows about.

#### 4.5.1 Event matrix

| ID | Event | Recipient | Channel | Timing | Class |
| --- | --- | --- | --- | --- | --- |
| N-01 | Request submitted | Manager | In-app + badge | Immediate | Core |
| N-02 | Request approved or rejected | Employee | In-app + badge | Immediate, includes decision note | Core |
| N-03 | Request still pending | Manager | In-app | After 2 working days, then daily | Core |
| N-04 | Request still pending and start date within 3 days | Manager and employee | In-app, flagged urgent | Daily until decided | Core |
| N-05 | Employee nudges approver | Manager | In-app | On demand, max once per 24 h per request, not on the day of submission | QoL |
| N-06 | Admin override or cancellation | Employee and original manager | In-app | Immediate, includes reason | Core |
| N-07 | Unused days expiring at year end | Employee | In-app | 30 days and 7 days before Dec 31 | QoL |
| N-08 | Daily or weekly digest of pending items | Manager, Admin | In-app (email unavailable locally) | Configurable | QoL (cut) |
| N-09 | Security events: password changed, repeated failed logins | Affected user | In-app | Immediate | QoL here; owned by the security ticket (ASVS V6.3.5, V6.3.7) |

#### 4.5.2 Rules that keep notifications sane

- **One open reminder per request per recipient.** Reminders are idempotent on (request, recipient, rule); a daily job must not stack copies (a queue of 14 identical rows is worse than silence).
- **No reminder to an approver who is themselves on approved leave.** Sage HR exposes exactly this suppression, and it is the difference between a thoughtful system and a nagging one.
- **Quiet hours and a single send time.** One scheduled run per day at a fixed time (for example 08:00) is enough; per-minute jobs are noise and cost.
- **The notification never contains more than the recipient may already see.** Type, dates and status only. The rejection reason travels to the employee (they are a party to it) but never to other employees. This is field-level authorization applied to messaging (ASVS V8.1.2), and it prevents the classic leak where a notification body bypasses a policy check.
- **Every notification links to a record the recipient can open.** If the link 403s, the notification system is teaching users that the app is broken; scope the query by the same policy used for the page (ASVS V8.2.2).
- **Reminder settings are admin-configurable** (days before first reminder, send time, on/off), defaulting to the values above, so the behaviour can be demonstrated without a code change.

#### 4.5.3 What "reminder" means for each role

- **Employee** is reminded of things they can act on: expiring days (N-07), a pending request nearing its start date (N-04), and the outcome of a decision (N-02).
- **Manager** is reminded of the one thing that blocks other people: an unactioned request. The escalation path is reminder (N-03) then urgency (N-04); at lab scale there is no third level, because the Admin already has the override (A-07).
- **Administrator** gets exception-driven signals, not a stream: requests stuck beyond the escalation window, overrides performed, and (if built) the daily digest.

#### 4.5.4 Mechanics on the recommended stack

- Laravel's database notifications give the inbox for free: a `notifications` table with `read_at`, `unreadNotifications`, `markAsRead`, and a badge count from `unreadNotifications()->count()` (Laravel 13 docs, notifications).
- Reminders are a scheduled command, not a queue worker: `Schedule::command(...)->dailyAt('08:00')` in `routes/console.php`, run locally with `php artisan schedule:work` (Laravel 13 docs, scheduling). Same mechanism runs the yearly reset (A-05).
- Do not build email. There is no reliable mail transport in a local-only lab deployment, and an SMTP dependency is the most likely cause of a failed demo (`research/tech-stack.md`, risk 4). Say "in-app notifications; email is a configured channel we did not enable" rather than pretending.
- Notification creation belongs in the service layer next to the state change (submit, decide, override, cancel), inside the same transaction, so a decision can never exist without its notification.

---

## 5. Priority ranking

### 5.1 Tiers

| Tier | Definition | Rule for the lab |
| --- | --- | --- |
| **P0 (core)** | A role cannot complete its job, or completes it wrongly, without this | Must be built; nothing else starts until these exist |
| **P1 (QoL, high value)** | High visibility or removes real friction at low cost | Build after P0 demos end-to-end |
| **P2 (QoL, low value)** | Cosmetic or situational | Explicitly deferred in the spec, not silently dropped |

### 5.2 Ranking by role

| Role | P0 core | P1 high-value QoL | P2 deferred |
| --- | --- | --- | --- |
| Employee | E-01, E-02, E-03, E-04, E-05, E-06, E-07, E-08 | E-09, E-11, E-12, E-10, E-13 | E-14, E-15, E-16 (conditional), E-17 |
| Manager | M-01, M-02, M-03, M-04, M-05, M-06 | M-08, M-07, M-11 | M-09, M-10, M-12 |
| Administrator | A-01, A-02, A-03, A-04, A-05, A-06, A-07, A-08 | A-09, A-10, A-11 | A-12, A-13, A-14, A-15 |
| Notifications | N-01, N-02, N-03, N-04, N-06 | N-05, N-07 | N-08 |
| Cross-cutting | X-03 (dates/days correctness), X-04 (lists), X-06 (confirmations), X-07 (flash messages) | X-01, X-02, X-05 | X-08, X-09 |

Counts: 22 P0, 15 P1, 9 P2 (E-16 is P1 if attachments are approved, otherwise P2).

### 5.3 Build order (dependency-aware)

1. **E-01 + E-02 + E-03 + E-04** - a person can request leave and see a balance.
2. **A-01 + A-03 + A-06** - roles, managers, leave types and holidays exist to request against.
3. **M-01 + M-02 + M-03 + M-04** - someone can decide, with context.
4. **E-05 + E-06 + M-06** - both sides can see and correct what happened.
5. **N-01 + N-02** - the loop closes without anyone leaving the app.
6. **A-02 + A-04 + A-05 + A-07 + A-08** - the admin can run the year and defend every decision.
7. **N-03 + N-04 + N-06** - the system chases what the humans forget.
8. Then P1 in any order, starting with E-11/M-08 (one calendar, two views) and E-09 (free timeline).

Steps 1-6 are the demo. Steps 7-8 are the difference between "works" and "feels finished".

### 5.4 What to cut first if time runs short

In order: X-08 dark mode, E-17 drafts, A-14 CSV import, M-09 bulk approve, N-08 digest, E-15 .ics export, A-10 reports, then M-10 clarification thread. **Never cut:** E-04 (pre-submit checks), M-04 (decision context), A-08 (audit log), N-02 (decision notification). Those four are the usability and credibility floor.

---

## 6. Rules and edge cases these features imply

The grilling ticket should approve these explicitly, because each one is a feature decision disguised as a detail:

1. **Self-approval is impossible.** A manager's own request routes to their manager; if they have none, to an Administrator. Otherwise M-02 and A-07 let a manager satisfy their own request.
2. **Routing uses the manager at submission time,** and the decision records `decided_by`. If the reporting line changes mid-flight, the request stays with the original approver unless an admin reassigns it.
3. **Whole days only in v1.** Half-days and hourly leave are a real product feature (Odoo models them explicitly) but they multiply the arithmetic, the balance model and the tests. Say "out of scope for v1" rather than half-building it.
4. **Past-dated requests are blocked** for employees; an Administrator can insert one with a reason. This keeps the demo honest about retroactive corrections.
5. **Any overlap with a non-rejected, non-cancelled request is blocked,** including pending ones. Two pending requests for the same day must not both be approvable.
6. **A request that would exceed the remaining balance is blocked server-side,** not merely warned. Admin adjustments may create a negative balance; employee requests may not.
7. **Cross-year requests** (for example Dec 30 to Jan 2) must have one rule: either blocked in v1, or counted entirely against the start year. Recommend blocking in v1 and documenting it.
8. **Deactivating a leave type hides it from new requests but keeps history readable.** Same for deactivated users: their requests remain visible to the admin and audit log, but they cannot sign in (ASVS V7.4.2).
9. **Cancellation of approved leave is admin-mediated** (E-12 raises a request; A-07 records the cancellation and restores the balance). Auto-cancel would give the employee a way to move their own balance after the fact.
10. **The rejection reason is visible to the employee and the admin, and to the deciding manager.** Not to other managers, and never inside a notification sent to anyone else.
11. **The last Administrator cannot be demoted or deactivated.** Otherwise the system can be locked out of its own configuration.
12. **Yearly reset is idempotent per (user, type, year).** A rerun of the job must not double the entitlement.

---

## 7. Screen inventory (input for the deferred wireframes ticket)

Thirteen screens cover the whole feature set; the first nine are the demo path.

| # | Screen | Serves |
| --- | --- | --- |
| 1 | Login, forgot/reset password | E-01, E-08 |
| 2 | Employee dashboard (balances, recent requests, unread notifications) | E-02, E-07 |
| 3 | Request form with inline checks | E-03, E-04 |
| 4 | My requests list and request detail with timeline | E-05, E-06, E-09 |
| 5 | Team calendar (availability) | E-10, E-11, M-08 |
| 6 | Manager approval queue | M-01, M-05 |
| 7 | Approval detail with decision context | M-02, M-03, M-04 |
| 8 | Admin users list and edit | A-01, A-02, A-15 |
| 9 | Admin leave types + holidays | A-03, A-06 |
| 10 | Admin balances and entitlement adjustment | A-04, A-05 |
| 11 | Admin audit log | A-08 |
| 12 | Reports and export | A-09, A-10, M-07 |
| 13 | Settings (reminders, session, policy) | A-11, A-12, N-03 |

---

## 8. Explicitly out of scope, and why

| Excluded | Why |
| --- | --- |
| Multi-level approval chains, delegation, "approve on behalf of" | The domain decision is single approval plus admin override; anything more is a different product and doubles the state machine (Odoo supports four approval modes precisely because it is a general HR suite). |
| Accrual plans (monthly/hourly earning), carry-over, expiry of rolled-over days | Already decided: yearly entitlement, reset yearly, no carry-over. Accrual is the largest single source of rules and the least valuable for a course demo. |
| Payroll, timesheet and calendar-sync integrations | No upstream systems in a local lab; every integration adds an external trust boundary for zero demo value. |
| Email delivery, SMS, push | No reliable transport locally; in-app notifications carry the whole story (`research/tech-stack.md`, risk 4). |
| Native mobile app | The responsive web view covers the use case (X-01). |
| SSO, MFA enrolment UI, password policy builder | Fortify provides the mechanisms; the security ticket owns which are enabled. Building a full identity console is a project on its own. |
| Impersonation ("view as user") | A demo convenience that creates a privilege-escalation path and an audit ambiguity; not worth it. |
| Half-day and hourly leave | See rule 3 above. |
| i18n / multi-language | No requirement, and it doubles every label on every screen. |
| Dark mode, keyboard shortcut layer | Cosmetics and power-user polish (X-08, X-09). |

---

## 9. Risks and open questions for the grilling ticket

1. **Attachments (E-16).** Keep the `requires_document` flag but ship only a reference field, or accept the file-upload surface (ASVS V5: type/size validation, storage outside the web root, download authorisation)? Recommendation: flag + reference field in v1.
2. **Half-days.** Confirm whole-days-only for v1; it is the single biggest scope lever in the feature list.
3. **Cross-year requests.** Block, or count against the start year? Recommendation: block and document.
4. **Withdrawal of approved leave.** Admin-mediated (recommended) or automatic with a notification? Affects E-12, A-07 and N-06.
5. **Team calendar visibility.** Same manager's team, or everyone? Recommendation: everyone authenticated, but name + type only, never the reason.
6. **Reminder defaults.** Two working days then daily, 08:00, with an admin toggle. Confirm the numbers so the spec, the demo script and the settings screen agree.
7. **Reports (A-10).** One exported report is enough for a rubric; three are wasted evenings. Recommendation: balances by person, CSV.
8. **Manager visibility of reasons.** Sick-leave reasons are sensitive; does a manager see the free-text reason for all types, or only for types flagged as "reason visible to approver"? Recommendation: visible to the direct approver, hidden from everyone else, and never in notification bodies.
9. **Admin visibility.** The Administrator sees everything by design, but that should be a stated decision with a log entry, not an accident of implementation.
10. **Notification overload.** If N-03/N-04 are on by default, the demo account will show a wall of reminders after a week of idle time; seed data must include a recent submission, and reminders must be idempotent.

---

## 10. Sources

First-party vendor documentation for feature vocabulary and behaviour:

- Odoo 18, Time Off overview: https://www.odoo.com/documentation/18.0/applications/hr/time_off.html (time off types, approval modes, allocations, public holidays, overview calendar with validated vs to-approve states, access-rights split between "My Time Off"/"Overview" and management).
- Odoo 18, Request time off: https://www.odoo.com/documentation/18.0/applications/hr/time_off/request_time_off.html (request fields, auto-computed duration, description, supporting document).
- Odoo 18, Management: https://www.odoo.com/documentation/18.0/applications/hr/time_off/management.html ("Waiting For Me" default filter, approve/refuse actions, status colouring).
- Odoo 18, Reporting: https://www.odoo.com/documentation/18.0/applications/hr/time_off/reporting.html (by employee, by type, balance pivot; reports insertable into a spreadsheet).
- Sage HR, "Set up email reminders for pending time off requests": https://support.sage.hr/en/articles/3515505-set-up-email-reminders-for-pending-time-off-requests (reminder days and time configuration, first reminder no earlier than the next day, suppression while the approver is away).
- SeamlessHR, "Leave Module Release Notes - Nudge Approver": https://support.seamlesshr.com/leave-module-release-notes-nudge-approver-version-1.1b-december-2025- (employee-initiated nudge, once per day per pending request, not available on the submission date).
- Nintex Automation, "Add a Reminder to the Manager Approval Step": https://help.nintex.com/en-US/nintexautomation/userguide/5.8/Content/Tutorials/Build/LeaveRequestExtended/07Escalation.htm (reminder fires before the leave start date if the manager has not responded).

Standards and framework documentation used to classify a feature as a control as well as a convenience:

- OWASP ASVS 5.0.0 (CSV release): https://github.com/OWASP/ASVS/tree/v5.0.0 - cited requirements: V1.2.10 (CSV formula injection), V2.2.2 and V2.3.1 (server-side validation and business-logic sequencing), V2.3.5 (multi-party scrutiny of sensitive flows), V6.2.2/V6.2.3 (users can change their own password), V6.3.5/V6.3.7 (notify on suspicious authentication and credential changes), V7.4.2/V7.4.5 (session termination on deactivation; admin session termination), V8.1.2 (field-level authorization), V8.2.1/V8.2.2 (function-level and data-level authorization), V16.2.1 (log metadata), V16.3.2/V16.3.3 (log failed authorization and control bypass).
- OWASP Top 10 2021, A09 Security Logging and Monitoring Failures: https://owasp.org/Top10/ (the audit log viewer as a user-visible control).
- W3C WCAG 2.2, Understanding SC 4.1.3 Status Messages: https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html (status messages must be programmatically determinable; informs X-02, X-07 and the notification UX).

Implementation mechanics:

- Laravel 13, Notifications: https://laravel.com/docs/13.x/notifications (database channel, `read_at`, `unreadNotifications`, `markAsRead`).
- Laravel 13, Task Scheduling: https://laravel.com/docs/13.x/scheduling (`Schedule::command(...)->dailyAt(...)`, `->yearlyOn(...)`, `php artisan schedule:work` for local development).
- Internal: `research/tech-stack.md` (recommended stack, notification/email risk, data model sketch with `leave_types.requires_document`, `notifications` table).
