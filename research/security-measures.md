# Security Measures & Testing Demo Plan

Research for the employee leave management system spec. Grounded in the OWASP Cheat Sheet Series (primary sources listed at the end). Sized for "defends against basic exploits" in a lab demo, not enterprise compliance. Framework-agnostic: every control has equivalent support in Laravel/Spring/Django/Express-style stacks, so the final choice can follow the stack ticket.

## Summary

The system needs roughly eleven security controls, each of which defeats a specific, demonstrable class of basic exploit: slow salted password hashing (Argon2id) defeats password-database cracking; framework-managed sessions with hardened cookies defeat session theft and fixation; server-side allow-list validation, parameterized queries, and output encoding defeat SQL injection and XSS; CSRF tokens defeat forged requests; deny-by-default role-based access control defeats privilege escalation and IDOR; login throttling defeats brute force; audit logging makes attacks visible; HTTPS protects credentials and session IDs in transit; and secure defaults (generic errors, security headers, least-privilege DB account) remove the free wins attackers try first. Every control can be demonstrated in a lab with a five-minute "attack, watch it fail, show the log" routine -- the demo plan below gives one per control, using only a browser, `curl`, and a database client. The strongest presentation structure is contrast: build one deliberately vulnerable page off to the side, exploit it live, then show the same attack failing against the real system.

## Part 1 -- Controls catalogue

Each control: the OWASP Top 10 2021 category it addresses, the threat/exploit it defeats, and a one-line implementation note.

| # | Control | OWASP category | Defeats (exploit) | Implementation note |
|---|---------|----------------|-------------------|---------------------|
| 1 | Password storage: Argon2id | A07 Identification & Authentication Failures | Offline cracking of a leaked password database (rainbow tables, dictionary/GPU brute force) | Hash with Argon2id (minimum m=19 MiB, t=2, p=1) using the framework's password hashing API; it salts automatically -- never store or invent a custom scheme. |
| 2 | Password policy & anti-enumeration login | A07 | Credential guessing, user enumeration via error messages | Enforce min length 8-15 chars, max 64+, check against a common-password blocklist; return one generic "Invalid email or password" for every login failure and always run the same code path (constant-time compare) so timing and wording never reveal whether the account exists. |
| 3 | Session management | A07 | Session hijacking (stolen/predicted session ID), session fixation | Use the framework's server-side session store; cookie must be `HttpOnly; Secure; SameSite=Lax` (or Strict) and named generically; regenerate the session ID on login; enforce an idle timeout (~30 min) and invalidate server-side on logout. |
| 4 | Input validation | A03 Injection | Malformed/malicious input reaching business logic (bad dates, negative durations, oversized strings) | Validate on the server with allow-list style checks (type, length, range, format) before any processing; validation complements, never replaces, query parameterization and output encoding. |
| 5 | SQL injection defence | A03 Injection | SQLi: attacker SQL executed via form fields or URL parameters (`' OR '1'='1`, stacked queries, data exfiltration) | Use parameterized queries / ORM parameter binding for every value; never concatenate user input into SQL; connect with a least-privilege DB account so even a bug can't DROP or read other tables. |
| 6 | XSS defence | A03 Injection (XSS) | Stored/reflected XSS: `<script>` in a comment or name field running in other users' browsers (can steal sessions, defeat CSRF) | Render all user data through the template engine's auto-escaping (never `raw`/`v-html`/`dangerouslySetInnerHTML` on user input); add a Content-Security-Policy header as a second layer. |
| 7 | CSRF defence | A01/CWE-352 (CSRF cheat sheet) | Forged cross-site requests: an attacker page submits "approve leave" or "change password" as the logged-in victim | Protect every state-changing request with the framework's built-in CSRF token (synchronizer token pattern for forms, custom header for API calls) plus `SameSite` cookies; never use GET for state changes. |
| 8 | Role-based access control (RBAC) | A01 Broken Access Control | Privilege escalation (employee opening admin URLs), IDOR (employee editing another employee's request) | Check role and ownership server-side on every request (deny by default; a hidden menu link is not authorization); object access must verify the record belongs to the caller (no trusting IDs from the URL). |
| 9 | Rate limiting / brute-force protection | A07 | Brute-force password guessing, credential stuffing, password spraying | Count failed logins per account (not per IP only) and throttle: e.g. exponential backoff or a short lockout after 5 failures within a window; apply per-IP rate limits to auth endpoints; log every lockout. |
| 10 | Audit logging & monitoring | A09 Security Logging & Monitoring Failures | Attacks happening invisibly (no way to detect/demonstrate brute force, IDOR, or approval abuse) | Server-side log of auth successes/failures, access-denied events, and approval/override actions with timestamp, user, and action; never log passwords, tokens, or full session IDs. |
| 11 | HTTPS / transport security | A02 Cryptographic Failures | Credential and session-cookie interception (mitM, sniffing) | Serve everything over TLS, even locally (self-signed cert for the demo); set the `Secure` cookie flag and HSTS; never send credentials over plain HTTP. |
| 12 | Secure defaults & configuration | A05 Security Misconfiguration (+ A06 for dependencies) | Verbose stack traces leaking internals, clickjacking, MIME-sniffing, default/dev credentials left on | Generic error pages, security headers (`X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, CSP, `Referrer-Policy`), secrets out of the repo, dependencies current; deny-by-default on all config. |

### Argon2id vs bcrypt -- the recommendation

**Recommend Argon2id.** Per the OWASP Password Storage Cheat Sheet, Argon2id (winner of the Password Hashing Competition) is the first choice for new systems: minimum configuration **m=19 MiB (19456 KiB), t=2, p=1**, and it is memory-hard, which makes GPU/cloud cracking expensive in a way bcrypt is not. Use **bcrypt (work factor >= 10)** only as the fallback if the chosen stack lacks a maintained Argon2id binding (bcrypt is the long-standing, extremely widely available option -- PHP's `password_hash`, Laravel, and most ecosystems support it by default). Both are acceptable answers to "defends against basic exploits"; the demo is identical for either. Practical notes: never use plain MD5/SHA-256 (too fast, crackable at scale); salts are handled by the library; a maximum password length of 64-72 bytes avoids bcrypt truncation and long-password DoS; a "pepper" (secret stored outside the DB) is optional defense-in-depth -- skip it for scope.

### Role model alignment (Employee / Manager / Administrator)

- **Employee**: own profile, own leave requests, own balance. Nothing else -- RBAC must enforce "own" (IDOR check), not just "authenticated".
- **Manager**: approve/reject requests of their direct reports. Must not administer leave types, users, or other managers' teams.
- **Administrator**: user management, leave-type configuration, override/cancel, audit log viewing. Must not be able to silently approve without the action being logged.
- Authorization should be a server-side policy layer ("can this role perform this action on this object?"), checked per request -- the single most demo-able control in the catalogue.

## Part 2 -- Testing / demo plan (basic-exploit lab presentation)

Setup: run the app locally, plus a DB client (to show the `users` table), `curl` (or PowerShell `Invoke-WebRequest`) for raw requests, and a throwaway `evil.html` page opened from a different origin (e.g. a second localhost port) for the CSRF demo. Optional but very effective for contrast: a `vulnerable-demo` page that skips the control being shown, so the audience sees the exploit work once, then fail against the real app. Each demo below is 2-5 minutes.

### Demo 1 -- Password storage (Control 1-2)

1. Register two users with the **same** password; open the `users` table in the DB client: two **different** hashes (salt working), `argon2id` visible in the PHC string.
2. Type the password somewhere; show it is nowhere in the DB (search all columns for the plaintext -- nothing). "The hash cannot be decrypted -- only guessed."
3. Optional contrast: a `vulnerable` table with `SHA-256(password)` or MD5; run a small wordlist through a script and crack it in seconds; show Argon2id resisting the same wordlist (each guess takes ~100 ms of memory-hard work).
4. Show the generic error: wrong password and nonexistent user produce the identical "Invalid email or password" message (no user enumeration).

### Demo 2 -- Session management (Control 3)

1. Log in as Employee; in DevTools show the session cookie flags: `HttpOnly`, `Secure`, `SameSite=Lax`. In the Console, run `document.cookie` -- the session cookie is absent (HttpOnly defeated a script from reading it).
2. Session hijacking: copy the cookie value; log out; replay the cookie with `curl` on a protected page -- redirected to login (server-side invalidation: deleting the cookie client-side is not enough; the server killed the session).
3. Session fixation: set a made-up session cookie in the browser, visit the login page, log in -- the server issued a **new** session ID (regeneration on privilege change).
4. Show the idle timeout: leave a session idle past the configured limit, next click returns to login.

### Demo 3 -- Input validation (Control 4)

1. In the leave-request form, submit: end date before start date, 0 or 999 days, and a 10,000-character comment. Each is rejected server-side with a specific, human-readable message (show it rejected even if the browser-side check is bypassed -- demonstrate via `curl` POST with a hand-crafted body that skips the UI).
2. Point: validation is server-side; what the browser allows is irrelevant to security.

### Demo 4 -- SQL injection (Control 5)

1. In the login form enter `anything' OR '1'='1' -- ` as the email. Login fails (treated as a literal string), vs. the vulnerable-demo page where the same input logs you in as the first user / dumps data.
2. Show the code side-by-side: concatenated query string (vulnerable) vs. parameterized query / ORM `where` clause (real app). The parameterized version never lets input become SQL syntax.
3. Optional: enable the DB query log and show the bound query -- the payload arrives as a quoted parameter value.

### Demo 5 -- XSS (Control 6)

1. Post a leave comment containing `<script>alert('xss')</script>`. View it: it renders as inert text; no alert. Inspect the page: the framework output-encoded it (`&lt;script&gt;`).
2. On the vulnerable-demo page, the same comment pops the alert for every viewer (stored XSS, session-stealable).
3. Show the CSP header in the response (DevTools > Network): even if encoding were bypassed, inline script is blocked.

### Demo 6 -- CSRF (Control 7)

1. While logged in as a Manager in the main app, open `evil.html` on the other origin: it auto-submits a form to the approve endpoint with a forged request. Show the server rejecting it (403) -- token missing/mismatch and/or `SameSite` cookie not sent cross-site.
2. Show the code: every state-changing form carries the CSRF token; removing the token from a real form and submitting gets the same 403.
3. Note the rule: state changes are never GET requests (show that `/approve?id=...` as GET does not exist).

### Demo 7 -- Role-based access control (Control 8)

1. Log in as Employee, paste an admin URL (`/admin/users`, `/admin/leave-types`) into the address bar: 403 "insufficient permissions" page -- not a redirect to a dashboard that happens to hide the links.
2. IDOR: as Employee A, request another employee's request by ID (`/leave-requests/42` where 42 belongs to B): rejected; also try changing the ID in the Approve button via a hand-crafted `curl` POST with Employee credentials. Rejected -- ownership is verified server-side against the session user.
3. Show the policy code: one server-side check per route ("role: manager AND request belongs to caller's team"), not a UI conditional.
4. Demo an Employee attempting to approve their own request via the API -- blocked by both role check and state machine (self-approval forbidden).

### Demo 8 -- Brute force / rate limiting (Control 9)

1. Run a loop of wrong-password logins against one account (curl in a `for` loop, ~20 attempts, wrong passwords from a wordlist). Show the server start throttling: after 5 failures, exponential backoff / temporary lockout with "Too many attempts, try again in N minutes".
2. Show it is per-account: from a "different IP" (spoof `X-Forwarded-For` in the curl loop if that header is trusted, or just explain) the lockout still holds.
3. Show the audit log recording the burst of failures and the lockout event (ties into Demo 9). The correct password still logs in after the window resets (no permanent DoS).

### Demo 9 -- Audit logging (Control 10)

1. Open the admin audit view after running demos 4-8: it already shows the failed logins, the lockout, the 403s, and the SQLi/XSS attempt payloads as rejected requests.
2. Perform one legitimate approval as Manager and one override as Admin; show both entries with timestamp, actor, action, target.
3. Show what is *not* logged: passwords and tokens never appear (grep the log for the password used in Demo 8 -- absent).

### Demo 10 -- HTTPS / transport (Control 11)

1. Serve the app with a self-signed TLS cert; show the padlock. Compare: on plain HTTP, open DevTools > Network, log in, and show the password visible in cleartext in the request (in the lab, a proxy or Wireshark on localhost makes the point); repeat over HTTPS -- encrypted.
2. Show the cookie with `Secure` and the HTTP-to-HTTPS redirect. (HSTS matters in production; mention it as a one-line config.)

### Demo 11 -- Secure defaults (Control 12)

1. Trigger an error (request a nonexistent request ID): generic "Something went wrong" page; no stack trace, no SQL, no framework version.
2. `curl -I` the app: show the security headers (`X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, CSP, `Referrer-Policy`).
3. Clickjack test: on the vulnerable-demo page that lacks `X-Frame-Options`, iframe the app and overlay a button (clickjacking); show the real app refusing to be framed.
4. Secrets: show `.env` in `.gitignore` -- DB credentials are not in the repository.

### Suggested presentation arc (10-12 min total)

1. Password storage (Demo 1) -- "even if the DB leaks, passwords are safe."
2. SQLi (Demo 4) and XSS (Demo 5) -- the two exploits everyone has heard of, shown blocked.
3. RBAC (Demo 7) -- privilege escalation and IDOR blocked, the domain-relevant control.
4. Brute force + audit log (Demos 8+9 together) -- attack is throttled *and* visible.
5. CSRF (Demo 6) and sessions (Demo 2) if time remains; HTTPS and secure defaults as fast closing shots.
6. Throughout: use the vulnerable-demo contrast page once per control (build it as throwaway pages, not in the shipped app).

## Explicitly out of scope (for this system)

- MFA/OIDC/passkeys (worth a "future work" slide, not an implementation item).
- Enterprise compliance (ISO 27001 / NIST audit trails), WAF, SIEM.
- Hashing upgrades/legacy migration (greenfield; Argon2id from day one).

## Sources (primary)

- OWASP Cheat Sheet Series: Password Storage (Argon2id m=19456/t=2/p=1; bcrypt work factor >= 10; hashing-not-encryption; salting/peppering) -- https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- OWASP Cheat Sheet Series: Authentication (password policy lengths per NIST SP 800-63B; generic error messages / anti-enumeration; login throttling & account lockout per-account; log all auth failures) -- https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Cheat Sheet Series: Session Management (>= 64 bits entropy, CSPRNG IDs; `HttpOnly`/`Secure`/`SameSite`; regenerate ID on privilege change; idle + absolute timeout; server-side invalidation on logout; never store tokens in localStorage) -- https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Cheat Sheet Series: SQL Injection Prevention (parameterized queries as primary defense; allow-list for identifiers/sort direction; least-privilege DB account; manual escaping strongly discouraged) -- https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series: Cross-Site Request Forgery Prevention (synchronizer token / signed double-submit; use framework built-ins; SameSite as defense-in-depth; no GET state changes; Fetch Metadata as modern option) -- https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series: Authorization (successor to the Access Control sheet: deny by default, centralized enforcement, IDOR/object-level checks) -- https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html and Insecure Direct Object Reference Prevention -- https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series: Cross Site Scripting Prevention (output encoding by context; safe template auto-escaping; CSP as layer 2) -- https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series: Logging (what to log: auth events, access control failures; what never to log: passwords, tokens) -- https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- OWASP Cheat Sheet Series: Transport Layer Security / HTTP Strict Transport Security (TLS everywhere, HSTS, Secure cookies) -- https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html
- OWASP Top 10:2021 (category mapping A01-A09) -- https://owasp.org/Top10/
- OWASP Web Security Testing Guide (manual test procedures used to shape the demos) -- https://owasp.org/www-project-web-security-testing-guide/
