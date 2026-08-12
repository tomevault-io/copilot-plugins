## ponko

> A self-hosted AI agent that lives in Slack. Single Go binary + Postgres.

# Ponko

A self-hosted AI agent that lives in Slack. Single Go binary + Postgres.

## Key Documentation

- **[docs/vision.md](docs/vision.md)** — Product vision and strategic direction
- **[docs/architecture.md](docs/architecture.md)** — Current system architecture (living document)
- **[docs/adr/](docs/adr/)** — Architecture Decision Records
- **[docs/slack-setup.md](docs/slack-setup.md)** — Slack bot setup guide

## Development Guidelines

- Target infrastructure cost: $3-15/month (excluding LLM usage)
- Single binary deployment (API + Workers)
- Postgres-backed job queue using River
- Deployment target: Fly.io (or any Docker host)

## Local Development

Key make targets: `make lint`, `make build`, `make test-unit`, `make test-e2e` (requires local Postgres), `make test-cover`

## CI/CD

- **CI** runs on every push to main and every PR: golangci-lint (v2.11.3 with fieldalignment checks), `make build`, `make test-unit`, `make test-e2e` (requires Postgres service container + ANTHROPIC_API_KEY)
- **CI build order**: Frontend must build before Go lint — `go:embed all:dist` in `web/embed.go` requires `web/dist/` to exist. The CI workflow handles this with a Node.js setup + `npm run build` step before lint.

### API Authentication

- `POST /api/workflows/start` — Bearer token auth only (programmatic access)
- `GET /api/workflows/{id}` — Bearer token OR session cookie (dual-auth: API clients and dashboard both work)
- `GET /api/workflows/recent` — Session cookie auth only (dashboard)
- `POST /api/jobs/{id}/discard` — Bearer token auth only (admin job cleanup)
- Health (`/health`) and Slack (`/slack/*`) endpoints are unauthenticated

- **Server-side env var**: `PONKO_API_KEY`
- **Client-side env var** (for making requests): `PONKO_AUTH_TOKEN`
- Usage: `Authorization: Bearer $PONKO_AUTH_TOKEN`

## E2E Tests

- 6 E2E test files share `setupE2EServer` in `internal/e2e/testhelpers_test.go` — only the helper needs updating when `NewServer` changes. `proactive_message_test.go` doesn't use the HTTP server and has its own setup.

## Code Quality Rules

- **No excessive comments**: Don't add comments that restate what code does. Only comment non-obvious "why" decisions.
- **No unnecessary abstractions**: Don't create interfaces, wrappers, or helpers for things used once. Three similar lines > premature abstraction.
- **No cargo-cult patterns**: Don't add retry logic where ops can't fail, don't memoize cheap operations, don't add feature flags for one-time changes.
- **Reuse existing code**: Before writing a new helper, search for existing utilities in the codebase.
- **Proper error handling**: Never silently swallow errors. Wrap with `fmt.Errorf` and return, or log with context. No empty error branches.
- **No over-engineering**: Don't add config objects for unchanging values, don't create abstract base types for single implementations, don't add generics unless needed.
- **No lint suppression as first resort**: For `fieldalignment`, reorder struct fields. For other lint rules, fix the code first. Only suppress with explicit user approval after exhausting code-level fixes.
- **Minimal test footprint**: Test behavior, not implementation details. Don't test framework behavior. Don't mock what you can call directly.

## Test-Driven Development

All new code follows TDD. Each red-to-green cycle gives unambiguous feedback.

### The TDD Loop

1. **Write failing tests first** based on expected input/output from acceptance criteria
2. **Run `make test-unit`** to confirm tests fail (compile errors count as failures)
3. **Commit the failing tests** as a checkpoint
4. **Implement minimum code** to make tests pass — do NOT modify the tests
5. **Run `make test-unit` again** to confirm green
6. **Iterate** until all tests pass, then refactor if needed

### Test Patterns

Test placement: `internal/foo/bar_test.go` alongside `internal/foo/bar.go`. Use `testutil.TestDB(t)` for database tests, `httptest.NewServer` for HTTP mocks. See `internal/user/store_test.go` and `internal/jobs/process_test.go` for patterns.

### River Worker Side Effects

River jobs that fail after producing side effects get retried, duplicating those side effects. Pattern:
- **Bad**: `createOutboxEntry()` then `transitionStatus()` — if transition fails, retry creates duplicate entries
- **Good**: Transition state first, then produce side effects — or make side effects idempotent

---
> Source: [edruder/ponko](https://github.com/edruder/ponko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
