# Tech stack comparison and recommendation

Research ticket: [#10](https://github.com/Wynteriall/leave-system/issues/10)
Map: [#1](https://github.com/Wynteriall/leave-system/issues/1)
Status: complete, ready for the grilling ticket.

---

## 1. Recommendation

**Laravel (PHP 8.3+ / Laravel 13) + MySQL 8, delivered as a single server-rendered monolith using Blade templates. No separate React SPA.**

One framework, one language, one codebase, one process to start. Every security control the course grades on is a framework default rather than custom code, which matters twice: less to build, and a cleaner story to present.

Why this wins on each weighted criterion:

| Criterion | How Laravel satisfies it |
| --- | --- |
| Ease of learning, use, presenting | One language end to end; `php artisan serve` gives a running app in minutes; routes, controllers, models and views map 1:1 onto how the team already reasons about a request. Middle difficulty: real framework, real ORM, real middleware, but no enterprise ceremony. |
| Familiarity | The team already studied PHP/Laravel **and** MySQL in college. Lowest ramp-up of any candidate; the security work starts on day one instead of week one. |
| Security maturity | Parameterized queries via Eloquent/PDO bindings; CSRF validation middleware enabled by default on the `web` route group; auth scaffolding with login, password reset, email verification, optional 2FA and login rate limiting (Fortify); password hashing with bcrypt/Argon2 through the `Hash` facade; Blade escapes output by default. |
| Frontend shape | Blade templates keep it one codebase, one deployment, one auth model (server-side sessions + CSRF tokens). Choosing the SPA route would add a second codebase and a token-based auth model for zero benefit in this scope. |

If the team later decides the UI must be React, the fallback is **Laravel + Inertia + React** (Laravel's official React starter kit): still one repository, still session auth, still Fortify, but it adds a JS build step and TypeScript/React work. That is a deliberate escalation, not the default path.

---

## 2. Weighted comparison

Weights reflect the brief: the stack has to be learned, built and defended in a lab, tonight to Nov 18.

- Ease of learning / use / understanding / presenting: **35%**
- Familiarity with the team's existing coursework: **30%**
- Out-of-the-box security maturity: **25%**
- Fit with the chosen frontend shape: **10%**

Scores 1-5 (5 = best).

| Criterion (weight) | Laravel + MySQL | Django + MySQL | Spring Boot + MySQL | ASP.NET Core + MySQL |
| --- | --- | --- | --- | --- |
| Ease / presenting (35%) | **5** | 3.5 | 2 | 2 |
| Familiarity (30%) | **5** | 2 | 4 | 1.5 |
| Security out of the box (25%) | **5** | 5 | 4.5 | 5 |
| Frontend shape fit (10%) | **5** | 4 | 3.5 | 4 |
| **Weighted total** | **5.00** | 3.48 | 3.38 | 2.80 |

Notes on the scores:

- **Familiarity**: Java was studied at "basics" level. Spring Boot is not Java basics; it is Java plus dependency injection, annotations, build tooling, JPA, and a security DSL. C# and ASP.NET Core appear nowhere in the team's history. PHP/Laravel plus MySQL is the only combination that is directly reused.
- **Ease**: Django would score higher if the team knew Python; the framework itself is about as approachable as Laravel. Spring Boot is penalized for configuration surface and boilerplate per feature. ASP.NET Core is penalized for tooling and C# ramp-up from zero.
- **Security**: all four are mature. The spread is small and comes from *how much is on by default and how visible it is to an audience*.

### 2.1 Candidate-by-candidate

#### Laravel (PHP + MySQL) - recommended

Pros

- Direct reuse of coursework: PHP syntax, Laravel structure, MySQL modeling.
- Security defaults are the ones the rubric asks about: CSRF middleware in the `web` group (`ValidateCsrfToken`), `@csrf` in Blade forms, Eloquent parameterized queries, `Hash::make` with bcrypt by default and Argon2/Argon2id selectable via `HASH_DRIVER`.
- Auth scaffolding is first-party: Fortify provides `/login`, `/logout`, `/forgot-password`, `/reset-password`, email verification, password confirmation and 2FA routes; login rate limiting is configurable (`RateLimiter::for('login', ...)`).
- Blade auto-escapes output, cutting XSS surface without extra libraries.
- Migrations + seeders make a repeatable, demo-safe local database (roles, leave types, entitlements seed in one command).
- Artisan gives a clean narrative for a demo: `migrate`, `db:seed`, `route:list`, `tinker`.

Cons

- PHP's loose typing and magic (facades, Eloquent conventions) can hide control flow from a reader who wants explicit structure.
- Less academically prestigious than Spring for a "security" course; the framework does a lot for you, which must be explained rather than shown as hand-written defense code.
- A local PHP toolchain (PHP, Composer, MySQL, optionally Docker for Sail) has to be installed on every team machine.

#### Django (Python + MySQL)

Pros

- Excellent documented security posture: querysets are parameterized, `CsrfViewMiddleware` is on by default, templates auto-escape, clickjacking middleware ships in the default settings, password hashing defaults to PBKDF2 with Argon2/bcrypt/scrypt available via `PASSWORD_HASHERS`.
- Django admin gives a free administrative back office, which maps neatly onto the Administrator role.
- Migration system and ORM are comparable to Laravel's.

Cons

- Python is not in the team's studied set; the framework would be learned alongside the language and its packaging/tooling.
- The free admin is double-edged: it is easy to demo, but it bypasses the custom role/authorization story the project is supposed to demonstrate.
- Django security guidance leans on deployment concerns (HTTPS redirect, secure cookies, HSTS) that do not apply to local dev, so part of the "security maturity" advantage is not demonstrable here.

#### Spring Boot (Java + MySQL)

Pros

- The team has Java basics, so syntax is not foreign.
- Spring Security is the reference implementation of the patterns being taught: filter chain, CSRF protection on by default for unsafe methods, `BCryptPasswordEncoder`/`DelegatingPasswordEncoder`, method-level authorization.
- Strong typing and explicit layering produce a very defensible architecture diagram.

Cons

- Steepest slope of the four: Maven/Gradle, starters, annotations, JPA/Hibernate mappings, `SecurityFilterChain` configuration, and a lot of code per feature.
- "Enterprise-heavy" is exactly what the brief excludes; most lab time would go into wiring rather than into leave-domain logic or security demonstration.
- SPA integration (if React were chosen) requires extra CSRF configuration for the SPA case in current Spring Security.

#### ASP.NET Core (C# + MySQL)

Pros

- Best-in-class integrated auth: ASP.NET Core Identity manages users, passwords, roles, claims and tokens; templates scaffold register/login/logout/change-password.
- Antiforgery is built into MVC/Razor form tag helpers (`__RequestVerificationToken`), and .NET 11 adds automatic CSRF middleware based on `Sec-Fetch-Site`/`Origin` by default.
- EF Core migrations and a mature DI container.

Cons

- Zero team familiarity: C#, .NET SDK, EF Core, Razor, and the ASP.NET middleware model all at once.
- Default templates and docs assume SQL Server/LocalDB; using MySQL means an extra provider and its quirks on the local dev path.
- The Identity defaults are broad and opinionated; explaining exactly which control blocks which OWASP category is harder than in Laravel or Django, where the surfaces are smaller.

---

## 3. Frontend shape: monolith vs SPA

The brief asks for both to be evaluated, noting the SPA adds a second codebase.

### Option A - Server-rendered monolith (recommended)

Laravel routes -> controllers -> Blade views. Session cookie auth, CSRF token in every form.

- **One codebase, one language, one `composer install`.**
- Auth is a session cookie (HttpOnly, SameSite=Lax) plus a CSRF token. No token storage decision, no refresh logic, no CORS.
- Validation lives once, server-side, in Form Requests; the same rules produce the error messages the view shows.
- Authorization is enforced where the data is read/written, so the "broken access control" demo is one middleware/policy away from the route.
- Fastest to build and to present: every screen is a route you can open directly.

Trade-off: page interactions are full requests or small amounts of vanilla JS; no client-side routing or optimistic UI.

### Option B - Separate React SPA + Laravel API

React app (own package.json, own dev server, own build) calling a JSON API authenticated with Sanctum.

- **Two codebases, two toolchains, two dependency trees, two start commands.**
- Auth shape changes: bearer tokens or Sanctum's cookie-based SPA mode; if tokens are used, storage and XSS exposure become a decision the team must defend.
- CSRF must be re-established for the API surface (Sanctum exists precisely for this), plus CORS configuration and a trusted-origin list.
- Validation is duplicated: server rules for integrity, client rules for UX, and they drift.
- More security surface to explain, and more places for the demo to fail on the day.

Trade-off: modern SPA UX and clean API separation, neither of which the brief requires.

| Aspect | Monolith (Blade) | Separate React SPA |
| --- | --- | --- |
| Codebases | 1 | 2 |
| Languages | PHP (+ minimal JS) | PHP + TypeScript |
| Auth | Session cookie + CSRF token | Sanctum cookie/SPA mode or bearer tokens |
| CSRF | Framework middleware, `@csrf` per form | Must be wired for API calls; token refresh on login/logout |
| CORS | Not needed (same origin) | Required, with an explicit trusted origin |
| Validation | One set of rules | Two sets, kept in sync by hand |
| Build step | Optional (Vite assets) | Mandatory |
| Demo risk | Low | Medium |
| Fits the brief | Yes | Over-scoped |

**Verdict:** monolith. The SPA's only real advantage (client-side interactivity) is not part of the feature set, while its costs (second codebase, token handling, CORS, duplicated validation) all land on the security story the project is graded on.

---

## 4. Proposed architecture

### 4.1 Layers

```
Browser
  |  HTTPS (local: optional) / HTTP on localhost
  v
+-------------------------------------------------------------+
| Presentation      Blade views, layouts, components,         |
|                   form partials, error bags                 |
+-------------------------------------------------------------+
| HTTP              Routes (web.php), controllers,            |
|                   Form Requests (validation), middleware    |
|                   (auth, role, CSRF, throttle)              |
+-------------------------------------------------------------+
| Domain / service  LeaveRequestService, BalanceService,      |
|                   ApprovalService, LeaveTypeService,        |
|                   NotificationService                       |
+-------------------------------------------------------------+
| Persistence       Eloquent models, query scopes,            |
|                   transactions, migrations, seeders         |
+-------------------------------------------------------------+
| Data              MySQL 8 (users, roles, leave_types,       |
|                   leave_balances, leave_requests,           |
|                   approvals, audit_log)                     |
+-------------------------------------------------------------+
```

Rules that keep the layering honest:

- Controllers do not query models directly for workflow decisions; they call a service.
- Services never read `request()`; they take plain arguments or DTOs. This is what makes the approval rules unit-testable.
- Authorization is checked twice: route-level middleware for coarse role gates (`role:manager`, `role:admin`) and policies for record-level checks (an employee may only see their own requests; a manager only their reports').

### 4.2 Data model sketch

| Table | Key columns | Notes |
| --- | --- | --- |
| `users` | id, name, email, password, role_id, manager_id, is_active | `password` holds a bcrypt/Argon2 hash; `manager_id` self-FK defines the reporting line |
| `roles` | id, name (employee/manager/admin) | Seeded, not user-editable |
| `leave_types` | id, name, annual_entitlement_days, requires_document, is_active | Admin-configurable; `is_active` instead of delete preserves history |
| `leave_balances` | id, user_id, leave_type_id, year, entitled_days, used_days | Unique on (user_id, leave_type_id, year); reset yearly, no carry-over |
| `leave_requests` | id, user_id, leave_type_id, start_date, end_date, days_requested, reason, status, decided_by, decided_at, decision_note | `status` in (pending, approved, rejected, cancelled) |
| `approvals` | id, leave_request_id, actor_id, action, note, created_at | Append-only trail of decisions and overrides |
| `audit_log` | id, actor_id, action, subject_type, subject_id, metadata(json), created_at | Login, role change, leave-type change, override, cancel |
| `notifications` | id, user_id, leave_request_id, type, read_at | In-app notifications; email optional and out of scope for local dev |

Derived rules worth writing down in the spec: `days_requested` is computed server-side from dates and a working-day calendar; overlap with existing non-rejected requests is rejected; approval decrements `used_days` in the same transaction that flips `status`; rejection leaves the balance untouched; admin override writes to `approvals` and `audit_log` and never deletes the original decision.

### 4.3 Auth and session flow

```
GET /login            -> login form (CSRF token embedded)
POST /login           -> validate, throttle, Auth::attempt
                         - regenerate session id on success
                         - redirect by role
GET /dashboard        -> auth middleware -> role-specific view
POST /leave-requests  -> auth + CSRF + Form Request + policy
POST /approvals/{id}  -> auth + CSRF + role:manager + policy(approve)
POST /admin/...       -> auth + CSRF + role:admin
POST /logout          -> auth + CSRF, invalidate session, regenerate token
```

Shape decisions:

- **Stateful sessions, not tokens.** Cookie is `HttpOnly`, `SameSite=Lax`, `Secure` only when served over HTTPS; lifetime short enough for a demo, long enough to not annoy.
- **Session fixation defense**: regenerate the session ID on login and invalidate on logout.
- **Password hashing**: `Hash::make`, bcrypt cost 12 (Laravel default is acceptable; Argon2id is the upgrade path and is a one-line `HASH_DRIVER` change).
- **Brute force**: Fortify's login rate limiter, 5 attempts per minute per email+IP; optional lockout window for the demo.
- **Password policy**: minimum length plus a check against common passwords; server-side validation only, mirrored in the form for UX.
- **Role checks**: middleware for route groups, policies for records. Never trust a hidden field or a client-side role flag.
- **2FA**: available via Fortify but optional; enable only if the demo needs a second factor story.

### 4.4 Local dev environment

Native path (simplest to present):

1. Install PHP 8.3+ with required extensions, Composer, MySQL 8, Node 20+ (only if Vite assets are customized).
2. `composer create-project laravel/laravel leave-system` (or `laravel new leave-system`).
3. `.env`: `DB_CONNECTION=mysql`, `DB_DATABASE=leave_system`, `DB_USERNAME`, `DB_PASSWORD`; `APP_KEY` generated by `php artisan key:generate`; `APP_DEBUG=true` locally only.
4. `php artisan migrate --seed` creates the schema plus roles, an admin, a manager, two employees and three leave types.
5. `php artisan serve` -> `http://127.0.0.1:8000`.

Container path (optional, if Docker is preferred): Laravel Sail. `php artisan sail:install` then `./vendor/bin/sail up`, which pins PHP and MySQL versions so every teammate has an identical environment.

Seeded demo accounts (documented in the spec, not committed with real credentials): `admin@example.test`, `manager@example.test`, `employee@example.test`.

Presentation notes:

- Keep `APP_DEBUG=false` for the "hardened" run so stack traces are not shown, then flip it deliberately to demonstrate why debug mode is a misconfiguration risk.
- Serve over HTTPS locally with `mkcert` if the demo needs `Secure` cookies and HSTS; otherwise state explicitly that cookie flags are relaxed for local HTTP only.
- `php artisan route:list --path=admin` is a good live proof that admin routes sit behind middleware.

### 4.5 Mapping to the security catalogue

| OWASP category | Control in this stack | How it is demonstrated |
| --- | --- | --- |
| A01 Broken access control | `role:` middleware + policies + scoped queries | Employee A opens Employee B's request URL -> 403; manager approves a request outside their reports -> 403 |
| A02 Cryptographic failures | `Hash` facade (bcrypt/Argon2id), HTTPS in demo | Show the `users.password` column contains a hash, never plaintext |
| A03 Injection | Eloquent bindings; no string-concatenated SQL | Submit `' OR 1=1 --` in a search/filter field -> treated as a literal, no rows leaked |
| A04 Insecure design | Server-side day calculation, overlap check, transactional balance update | Attempt to request more days than the remaining balance -> rejected server-side even if the form is tampered with |
| A05 Security misconfiguration | `APP_DEBUG=false`, `.env` not committed, secure headers | Trigger a 500 with debug off -> generic error page |
| A07 Auth failures | Fortify throttling, session regeneration, password policy | Six failed logins -> 429; inspect regenerated session cookie across login |
| A08 Data integrity | `composer.lock` committed, signed releases | Show lockfile and `composer audit` |
| A09 Logging failures | `audit_log` table + Laravel log channel | Admin override appears in audit view |
| A10 SSRF | No user-supplied outbound URLs in scope | State as a non-applicable surface with justification |

CSRF is worth calling out separately from the OWASP numbering: Laravel's `ValidateCsrfToken` middleware rejects a POST without a matching token, and removing the hidden `_token` from a form in dev tools reproduces a 419 on the spot. That is the single most convincing live demo in the whole catalogue.

---

## 5. Risks and open points for the grilling ticket

1. **PHP toolchain friction on Windows.** PHP extensions and MySQL on Windows are the most likely setup failure. Sail removes this risk at the cost of Docker; decide before build starts.
2. **"You did not write the security yourself" objection.** Mitigation: for each control, show the framework mechanism, the configuration line that enables it, and one failing request that proves it is active.
3. **React expectations.** If another course deliverable requires a React artefact, do not merge requirements; keep this system a monolith and note Inertia as the sanctioned upgrade path.
4. **Scope creep via notification email.** Local dev has no reliable mail transport; keep notifications in-app and state that email is out of scope.
5. **Testing story.** Feature tests with Laravel's HTTP test client cover role gates, CSRF-off behavior in tests, and balance arithmetic. The security demo plan should name concrete test cases, not just manual clicks.

---

## 6. Sources

Primary documentation consulted (all first-party):

- Laravel, CSRF protection: https://laravel.com/docs/12.x/csrf (middleware in the `web` group, `@csrf`, `X-CSRF-TOKEN`, SPA note pointing to Sanctum)
- Laravel, Starter kits and Fortify routes: https://laravel.com/docs/13.x/starter-kits
- Laravel, Hashing: https://laravel.com/docs/13.x/hashing (bcrypt default, Argon2/Argon2id, `Hash::make`, `needsRehash`)
- Django, Security in Django: https://docs.djangoproject.com/en/5.1/topics/security/ (parameterized querysets, `CsrfViewMiddleware`, template auto-escaping, clickjacking middleware)
- Django, Password management: https://docs.djangoproject.com/en/stable/topics/auth/passwords/ (PBKDF2 default, Argon2/bcrypt/scrypt, validators)
- Spring Security, CSRF: https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html (on by default for unsafe methods, SPA requires extra configuration)
- ASP.NET Core, Prevent XSRF/CSRF: https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery (antiforgery tokens, automatic CSRF middleware in .NET 11)
- ASP.NET Core, Introduction to Identity: https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity (users, passwords, roles, claims, scaffolding)
- OWASP Top 10: https://owasp.org/Top10/

Version note: Laravel 13.x is current at the time of writing; Django 6.1 and Spring Security 7.1 are current; ASP.NET Core documentation defaults to .NET 10 with .NET 11 notes. The security mechanisms cited are stable across the versions named.
