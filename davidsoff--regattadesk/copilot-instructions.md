## regattadesk

> This file defines best practices for human and LLM agents working in this repository.

# AGENTS.md

This file defines best practices for human and LLM agents working in this repository.

## Mission
Build RegattaDesk v0.1 in small, testable increments while preserving correctness, operability, accessibility, and clear documentation.

## Source of truth
Always align changes with these artifacts before implementation:
- `pdd/implementation/plan.md`
- `pdd/implementation/bc*.md`
- `pdd/implementation/issues/*.issues.yaml`
- `pdd/design/openapi-v0.1.yaml`
- `pdd/design/style-guide.md`
- `pdd/design/database-schema.md`

If implementation and docs diverge, update docs in the same change.

## Working style
- Keep changes scoped and reversible.
- Prefer incremental, demoable outcomes over large batch changes.
- Use explicit assumptions; do not hide uncertainty.
- Do not introduce speculative abstractions without immediate use.
- RegattaDesk v0.1 is pre-production: breaking changes are allowed when they simplify code and remove unused migration/deprecation paths.
- When introducing a breaking change, update affected PDD docs in the same change.

## Planning and execution
- Start from a single ticket (or one tightly related ticket set).
- Identify dependencies (`depends_on`) before coding.
- Define acceptance criteria and tests up front.
- Finish one concern fully (code + tests + docs) before moving on.

## Local command reference
- Install dependencies: `make install`
- Build all apps: `make build`
- Run CI-equivalent checks: `make test`
- Run lint only: `make lint`
- Backend unit tests only: `cd apps/backend && ./mvnw test`
- Frontend lint only: `cd apps/frontend && npm run lint`
- Regenerate frontend API client: `cd apps/frontend && npm run api:generate`

## Architecture guardrails
- Follow API-first development for staff and operator surfaces.
- Treat event history as append-only; do not mutate audit/event records.
- Exception: BC06 line-scan tile storage is intentionally not event-sourced in v0.1; tile binaries and immediate storage metadata are API-managed persistence.
- Keep revision-driven public delivery semantics (`draw_revision`, `results_revision`) intact.
- Keep auth boundaries explicit: edge identity via Traefik/Authelia, API role enforcement in backend, and operator token rules for line-scan ingest/retrieval.
- Respect bounded-context ownership; avoid leaking domain logic across contexts.

## Coding best practices
- Prefer clear, boring code over clever code.
- Keep functions small and side effects explicit.
- Validate all external input at boundaries.
- Fail fast with actionable errors.
- Use deterministic behavior for workflows requiring replay/recovery.
- Prefer idempotent command handling for retry-prone paths.
- Keep configuration centralized and environment-driven.

### Bash scripting
- Use `[[` instead of `[` for conditional tests (safer and more feature-rich).
- Quote variables to prevent word splitting and glob expansion.
- Use `set -e` for scripts that should fail on first error.
- Prefer explicit error handling over silent failures.

## Data and persistence
- Use migrations for all schema changes.
- Define indexes with observed query patterns in mind.
- Encode invariants in both domain logic and DB constraints where practical.
- Never silently drop or rewrite historical business events.

## API and contracts
- Treat OpenAPI and Pact contracts as enforceable interfaces.
- Add contract tests when request/response behavior changes.
- Version behavior intentionally; avoid accidental contract drift.
- Return stable, machine-readable error shapes.

## Frontend and UX
- Reuse shared design tokens and primitives.
- Meet WCAG 2.2 AA minimum for impacted flows.
- Ensure keyboard navigation and focus states are usable.
- Keep locale/timezone/date formatting consistent with PDD rules.
- Validate mobile and desktop behavior for user-facing flows.

## Security and privacy
- Enforce least privilege by role.
- Apply secure defaults for cookies, headers, and token handling.
- Never commit secrets, tokens, or credentials.
- Log security-relevant actions with actor/context metadata.

## Testing standards
Apply the test matrix from `pdd/implementation/plan.md`:
- Domain/command logic: unit tests.
- Persistence/query changes: PostgreSQL integration tests.
- API contract changes: Pact + endpoint integration checks.
- UI/UX changes: targeted UI tests + accessibility checks.

Also:
- Add regression tests for bug fixes.
- PR-gating tests must be deterministic and environment-stable.
- Do not gate PRs on timing thresholds, `sleep`-based checks, wall-clock assumptions, or machine-local timezone/locale behavior.
- Keep performance benchmarks/sanity checks non-gating and isolated from default PR test jobs.
- Keep test fixtures readable and minimal.

## Observability and operations
- Add metrics/tracing for new critical paths.
- Keep health/readiness signals meaningful.
- Document operational impacts and runbook updates in the same PR when relevant.
- Treat performance regressions as functional defects.

## Documentation and handoff
For each meaningful change:
- Update relevant PDD docs and ticket status/context.
- Document migration steps and rollback notes when relevant to an actively used environment.
- Include concise "what changed" and "how to verify" notes.

## Git and commit hygiene
- Make atomic commits with one primary concern per commit.
- Use descriptive commit messages with clear intent.
- Do not mix refactors with behavior changes unless necessary.
- Keep the working tree clean before handoff.

## Pull request checklist
- Scope matches ticket(s).
- Acceptance criteria satisfied.
- Required tests added and passing.
- Docs/contracts updated.
- Security, accessibility, and operability impacts considered.
- Risks and follow-ups explicitly listed.

## Anti-patterns to avoid
- Hidden behavior changes without tests.
- Silent schema changes without migrations.
- Cross-context coupling for short-term convenience.
- TODO-driven shipping without linked tickets.
- Disabling tests or checks to make CI pass.

## Issue Tracking

This project uses **bd (beads)** for issue tracking.
Run `bd prime` for workflow context, or install hooks (`bd hooks install`) for auto-injection.

**Quick reference:**
- `bd ready` - Find unblocked work
- `bd create "Title" --type task --priority 2` - Create issue
- `bd close <id>` - Complete work
- `bd dolt push` - Push beads to remote

For full workflow details: `bd prime`

<!-- BEGIN BEADS INTEGRATION v:1 profile:full hash:d4f96305 -->
## Issue Tracking with bd (beads)

**IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Why bd?

- Dependency-aware: Track blockers and relationships between issues
- Git-friendly: Dolt-powered version control with native sync
- Agent-optimized: JSON output, ready work detection, discovered-from links
- Prevents duplicate tracking systems and confusion

### Quick Start

**Check for ready work:**

```bash
bd ready --json
```

**Create new issues:**

```bash
bd create "Issue title" --description="Detailed context" -t bug|feature|task -p 0-4 --json
bd create "Issue title" --description="What this issue is about" -p 1 --deps discovered-from:bd-123 --json
```

**Claim and update:**

```bash
bd update <id> --claim --json
bd update bd-42 --priority 1 --json
```

**Complete work:**

```bash
bd close bd-42 --reason "Completed" --json
```

### Issue Types

- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `chore` - Maintenance (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (default, nice-to-have)
- `3` - Low (polish, optimization)
- `4` - Backlog (future ideas)

### Workflow for AI Agents

1. **Check ready work**: `bd ready` shows unblocked issues
2. **Claim your task atomically**: `bd update <id> --claim`
3. **Work on it**: Implement, test, document
4. **Discover new work?** Create linked issue:
   - `bd create "Found bug" --description="Details about what was found" -p 1 --deps discovered-from:<parent-id>`
5. **Complete**: `bd close <id> --reason "Done"`

### Auto-Sync

bd automatically syncs via Dolt:

- Each write auto-commits to Dolt history
- Use `bd dolt push`/`bd dolt pull` for remote sync
- No manual export/import needed!

### Important Rules

- ✅ Use bd for ALL task tracking
- ✅ Always use `--json` flag for programmatic use
- ✅ Link discovered work with `discovered-from` dependencies
- ✅ Check `bd ready` before asking "what should I work on?"
- ❌ Do NOT create markdown TODO lists
- ❌ Do NOT use external issue trackers
- ❌ Do NOT duplicate tracking systems

For more details, see README.md and docs/QUICKSTART.md.

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd dolt push
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

<!-- END BEADS INTEGRATION -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Davidsoff) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
