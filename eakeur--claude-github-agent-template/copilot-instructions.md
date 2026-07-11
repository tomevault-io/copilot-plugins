## claude-github-agent-template

> This file is read by Claude Code on every automated run. It defines standards,

# AGENTS.md

This file is read by Claude Code on every automated run. It defines standards,
conventions, and constraints that apply to ALL agentic workflows in this repo.

---

## Stack

- **Language**: Go
- **Infrastructure**: AWS Lambda + API Gateway + CloudFront + S3
- **Database**: PostgreSQL via Supabase
- **IaC**: Terraform
- **Query layer**: sqlc
- **CI/CD**: GitHub Actions

---

## Coding standards

- Follow standard Go project layout (`cmd/`, `internal/`, `pkg/`).
- All exported functions must have godoc comments.
- Error wrapping: use `fmt.Errorf("context: %w", err)` — never swallow errors.
- No `panic` in library code; only in `main()` for unrecoverable init failures.
- All DB queries must go through sqlc-generated code — no raw `database/sql` calls outside of migrations.
- Use `context.Context` as the first parameter on every function that does I/O.
- Tests live next to the code they test (`foo_test.go`). Table-driven tests preferred.
- Run `go vet ./...` and `staticcheck ./...` before committing.

---

## Git conventions

- Branch naming: `feat/<issue-number>-<slug>`, `fix/<issue-number>-<slug>`, `chore/<slug>`
- Commit messages follow Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`
- Every PR must reference its issue: `Closes #<n>` in the PR body.
- Never force-push to `main` or `develop`.
- Squash-merge preferred for feature branches; merge commits for release branches.

---

## PR standards

- PR titles follow Conventional Commits format.
- Description must include: **What**, **Why**, **How to test**.
- All new endpoints require OpenAPI spec update in `docs/openapi.yaml`.
- All DB schema changes require a new migration file in `db/migrations/`.
- Breaking changes require an ADR in `docs/adrs/`.
- Minimum one human approval before merge — Claude-opened PRs are not self-mergeable.

---

## Database conventions

- Migration files: `db/migrations/YYYYMMDDHHMMSS_<description>.sql`
- Always provide both `up` and `down` migrations.
- Never drop columns in a single migration — deprecate then remove in a later PR.
- Row-level locking: use `SELECT ... FOR UPDATE SKIP LOCKED` for job queue patterns.
- Never use `SELECT *` in sqlc queries.

---

## OpenAPI conventions

- Spec lives at `docs/openapi.yaml` (OpenAPI 3.1).
- All request/response bodies must have `$ref` schemas, not inline definitions.
- Error responses use the shared `#/components/schemas/ErrorResponse` schema.
- Version the API in the path: `/v1/...`

---

## ADR conventions

- ADRs live in `docs/adrs/` as `NNNN-<slug>.md`.
- Use the template at `docs/adrs/TEMPLATE.md`.
- An ADR is required whenever: a new external dependency is added, an existing
  architectural pattern is changed, or a breaking API/DB change is introduced.
- ADR status values: `proposed` → `accepted` → `deprecated` / `superseded by NNNN`.

---

## README conventions

- Keep `README.md` at the root up to date with: setup instructions, env vars,
  how to run locally, how to run tests, and architecture overview.
- Service-level READMEs live in `cmd/<service>/README.md`.

---

## Security constraints

- Never commit secrets, API keys, or credentials — use AWS Secrets Manager or
  GitHub Actions secrets.
- Never log request bodies that may contain PII.
- All Lambda functions must have least-privilege IAM roles defined in Terraform.

---

## What Claude should NOT do autonomously

- Merge PRs (always leave for human review).
- Delete branches on `main` or `develop`.
- Modify Terraform state files directly.
- Change IAM policies or security group rules without a human-reviewed PR.
- Close issues without confirmation (label `needs-human` instead).

---
> Source: [eakeur/claude-github-agent-template](https://github.com/eakeur/claude-github-agent-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
