## 05-security

> Security gate. Applied alongside verification gates on every task.


# Security gate

Builders run these checks before marking `in-review`.
QA runs these checks as part of review.
A task cannot move to `done` with any unresolved security finding.

## Check 1 — Secret scanning

Scan every file in `Files to create/modify` for:
- hardcoded API keys, tokens, passwords, private keys
- `.env` values committed to source (including `.env.local`, `.env.production`)
- connection strings with embedded credentials
- cloud provider access keys (AWS/GCP/Azure patterns)

Preferred tools (use whichever is available in the project):
- `gitleaks detect --no-git --source .`
- `trufflehog filesystem . --only-verified`
- `detect-secrets scan`

If no secret scanner is configured, Builder installs one during the scaffold
task and records the decision in `memory/decisions.md`.

## Check 2 — Dependency audit

Run the stack's dependency audit tool on any task that touches dependency
manifests (`package.json`, `pyproject.toml`, `requirements.txt`, `go.mod`,
`Cargo.toml`, `Gemfile`).

- npm/pnpm/yarn: `npm audit --audit-level=high`
- pip: `pip-audit`
- go: `govulncheck ./...`
- cargo: `cargo audit`

Zero `high` or `critical` advisories is required. Any accepted exception is
recorded with rationale in `memory/decisions.md` under a `Security exception`
heading.

## Check 3 — Auth boundary integrity

For any task that adds or modifies routes, handlers, or screens:

- Every protected route/handler must invoke the project's canonical auth
  middleware or session check. No route can rely on "assumed to be protected
  by parent router" without an explicit assertion.
- Public routes are listed explicitly in the task's acceptance criteria with
  a rationale.
- Test suite includes at least one test per protected route that asserts
  `401` (or project equivalent) for missing/invalid credentials.

## Check 4 — Input validation at boundaries

- Every API handler validates request input (body/query/params) with the
  project's validation library (Zod, Pydantic, class-validator, etc.) before
  calling service code.
- Every form field is validated before submission.
- Database queries are parameterized. Raw concatenated SQL is rejected.

## Check 5 — Output hygiene

- No stack traces, internal error messages, or raw DB errors returned to the
  client. Error shape must match the project's NFR contract.
- No PII or secrets written to logs (log statements reviewed for user data).
- Tokens never logged, never rendered in HTML, never embedded in URLs.

## Check 6 — CSP / CORS / headers (web UI tasks)

For any task touching server response headers or web deployment config:
- CORS origin allowlist is explicit — no `*` in production paths.
- CSP is present or a `Security: CSP deferred to TASK-XXX` note exists.
- Cookie flags: `HttpOnly`, `Secure`, `SameSite` are set where applicable.

## Rejection handling

If any check fails:
- Builder: fix before handoff. Do not mark `in-review`.
- QA: reject with `needs-fix`. Rejection note must cite the specific file,
  line, finding, and expected fix.

## Rule of thumb

When a security concern is ambiguous, choose the stricter interpretation and
surface the decision to the user in the QA notes field. Never silently relax
a security default.

---
> Source: [BolderApps/agentic-workflow](https://github.com/BolderApps/agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
