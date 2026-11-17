# ADR-0007: Supporting-document upload — real files, non-public storage, policy-gated download

Date: 2026-11-17 · Status: Accepted · Approved in #11 (professor-spec alignment) · Supersedes the
E-16 reference-field decision made in #6

## Context

The professor's reference (`docs/employee_leave_application_system.md`) names supporting-document
upload three times: the Employee Module ("upload supporting documents when required"), the User
Roles table, and Security Requirement 6 ("validate … uploaded documents"). The approved spec
(#6) downgraded E-16 to a document-*reference* field to avoid the file-handling surface
(ASVS V5). That deviation is directly against a graded requirement, so the decision was reopened
in #11.

For an Information Assurance project the upload surface is also an opportunity: file upload is a
canonical attack class (spoofed extension, path traversal, direct-serving of payloads), so the
feature doubles as another "attack, watch it fail" demo rather than a liability.

## Decision

1. **Real upload, scoped tight.** A file input on the request form (E-03) only when the chosen
   leave type has `requires_document = TRUE`.
2. **Validation in the Form Request (extends C-4):** PDF/JPEG/PNG only, matched by MIME **and**
   extension, maximum 5 MB, validated server-side before any processing.
3. **Storage (extends C-12):** Laravel non-public disk (`storage/app`), random generated
   filename, never executable, never served directly by the web server.
4. **Download authorization (extends C-8):** a dedicated download route behind the request
   policy — the employee owner, the direct approver, and Administrators only. Download events
   are logged in the audit log (data-minimisation criterion, §8.2).
5. **Schema:** a small `attachments` table (`leave_request_id`, `original_name`, `stored_path`,
   `mime`, `size`, `uploaded_by`, `uploaded_at`); one file per request while pending,
   replaceable. The §7.2 "not modelled: attachments" note is removed.

## Consequences

- The graded requirement is met and inspectable in a demo (rejected renamed `.php` upload,
  logged download attempt).
- File-handling surface exists but is bounded by the whitelist, the size cap, non-public
  storage, and the policy-gated route.
- `attachments` rows follow the request lifecycle; deleting a request (never done in v1 —
  append-only) would need a storage-cleanup story only if that ever changes.

## Alternatives considered

- **Keep the reference field (#6 decision)** — cheapest, but contradicts the reference in three
  places, including a security requirement; indefensible at grading.
- **Full multi-file upload with versioning** — more realistic, but a larger surface than the
  requirement needs; one file per request keeps the demo narrow.
- **Public disk + signed URLs** — simpler serving, but fails the data-privacy posture; files
  must only be reachable through the authorization policy.
