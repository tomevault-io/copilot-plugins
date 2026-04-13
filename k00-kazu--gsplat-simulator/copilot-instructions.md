## gsplat-simulator

> This is a Gaussian splatting rendering control engine for simulations.

# gsplat-simulator

This is a Gaussian splatting rendering control engine for simulations.

## Core Principles

1. **Agent-First** — Delegate to specialized agents for domain tasks
2. **Test-Driven** — Write tests before implementation, 80%+ coverage required
3. **Security-First** — Never compromise on security; validate all inputs
4. **Immutability** — Always create new objects, never mutate existing ones
5. **Plan Before Execute** — Plan complex features before writing code

## Available Agents

**difinision folder:** repo-root/.codex/agents

## Agent Orchestration

Use agents proactively without user prompt:
- Complex feature requests → **planner**
- Code just written/modified → **code-reviewer**
- Bug fix or new feature → **tdd-guide**
- Architectural decision → **architect**
- Security-sensitive code → **security-reviewer**
- Autonomous loops / loop monitoring → **loop-operator**
- Harness config reliability and cost → **harness-optimizer**

Use parallel execution for independent operations — launch multiple agents simultaneously.

## Security Guidelines

**Before ANY commit:**
- No hardcoded secrets (API keys, passwords, tokens)
- All user inputs validated
- SQL injection prevention (parameterized queries)
- XSS prevention (sanitized HTML)
- CSRF protection enabled
- Authentication/authorization verified
- Rate limiting on all endpoints
- Error messages don't leak sensitive data

**Secret management:** NEVER hardcode secrets. Use environment variables or a secret manager. Validate required secrets at startup. Rotate any exposed secrets immediately.

**If security issue found:** STOP → use security-reviewer agent → fix CRITICAL issues → rotate exposed secrets → review codebase for similar issues.

## Coding Style

**Immutability (CRITICAL):** Always create new objects, never mutate. Return new copies with changes applied.

**File organization:** Many small files over few large ones. 200-400 lines typical, 800 max. Organize by feature/domain, not by type. High cohesion, low coupling.

**Error handling:** Handle errors at every level. Provide user-friendly messages in UI code. Log detailed context server-side. Never silently swallow errors.

**Input validation:** Validate all user input at system boundaries. Use schema-based validation. Fail fast with clear messages. Never trust external data.

**Code quality checklist:**
- Functions small (<50 lines), files focused (<800 lines)
- No deep nesting (>4 levels)
- Proper error handling, no hardcoded values
- Readable, well-named identifiers

**Files typically involved**:
- `docs/rules/*/coding-style.md`
- `docs/rules/*/hooks.md`
- `docs/rules/*/patterns.md`
- `docs/rules/*/security.md`
- `docs/rules/*/testing.md`

## Testing Requirements

**Minimum coverage: 80%**

Test types (all required):
1. **Unit tests** — Individual functions, utilities, components
2. **Integration tests** — API endpoints, database operations
3. **E2E tests** — Critical user flows

**TDD workflow (mandatory):**
1. Write test first (RED) — test should FAIL
2. Write minimal implementation (GREEN) — test should PASS
3. Refactor (IMPROVE) — verify coverage 80%+

Troubleshoot failures: check test isolation → verify mocks → fix implementation (not tests, unless tests are wrong).

## Development Workflow

1. **Plan** — Use planner agent, identify dependencies and risks, break into phases
2. **TDD** — Use tdd-guide agent, write tests first, implement, refactor
3. **Review** — Use code-reviewer agent immediately, address CRITICAL/HIGH issues
4. **Capture knowledge in the right place**
   - Personal debugging notes, preferences, and temporary context → auto memory
   - Team/project knowledge (architecture decisions, API changes, runbooks) → the project's existing docs structure
   - If the current task already produces the relevant docs or code comments, do not duplicate the same information elsewhere
   - If there is no obvious project doc location, ask before creating a new top-level file
5. **Commit** — Conventional commits format, comprehensive PR summaries

## Workflow Surface Policy

- `skills/` is the canonical workflow surface.
- New workflow contributions should land in `skills/` first.
- `commands/` is a legacy slash-entry compatibility surface and should only be added or updated when a shim is still required for migration or cross-harness parity.

## Git Workflow

**Commit format:** `<type>: <description>` — Types: feat, fix, refactor, docs, test, chore, perf, ci

**PR workflow:** Analyze full commit history → draft comprehensive summary → include test plan → push with `-u` flag.

## Architecture Patterns

**API response format:** Consistent envelope with success indicator, data payload, error message, and pagination metadata.

**Repository pattern:** Encapsulate data access behind standard interface (findAll, findById, create, update, delete). Business logic depends on abstract interface, not storage mechanism.

**Skeleton projects:** Search for battle-tested templates, evaluate with parallel agents (security, extensibility, relevance), clone best match, iterate within proven structure.

## Performance

**Context management:** Avoid last 20% of context window for large refactoring and multi-file features. Lower-sensitivity tasks (single edits, docs, simple fixes) tolerate higher utilization.

**Build troubleshooting:** Use build-error-resolver agent → analyze errors → fix incrementally → verify after each fix.

## Project Structure

```
project-root/
   ├─ README.md
   ├─ LICENSE
   ├─ .gitignore
   ├─ run.py
   ├─ .python-version
   ├─ rust-toolchain.toml
   ├─ configs/
   │  ├─ app.toml
   │  ├─ transport.toml
   │  ├─ logging.toml
   │  └─ scenes/
   │     └─ sample_scene.toml
   ├─ third_party/
   │  └─ qt/
   │     ├─ .gitkeep
   │     ├─ windows/
   │     └─ ubuntu/
   ├─ runtime/
   │  ├─ logs/
   │  │  ├─ core/
   │  │  ├─ render_worker/
   │  │  └─ ui/
   │  ├─ ipc/
   │  ├─ cache/
   │  └─ doctor/
   ├─ assets/
   │  ├─ sample_models/
   │  └─ README.md
   ├─ apps/
   │  ├─ ui/
   │  │  ├─ CMakeLists.txt
   │  │  ├─ src/
   │  │  ├─ include/
   │  │  └─ resources/
   │  ├─ core/
   │  │  ├─ Cargo.toml
   │  │  └─ src/
   │  └─ render/
   │     ├─ requirements.txt
   │     ├─ src/
   │     ├─ tests/
   │     └─ .venv/
   ├─ crates/
   │  ├─ shared_schema/
   │  ├─ config/
   │  ├─ transport/
   │  └─ ipc_protocol/
   ├─ docs/
   │  ├─ build.md
   │  ├─ run.md
   │  ├─ environment.md
   │  ├─ qt_runtime.md
   │  └─ doctor.md
   ├─ tests/
   │  ├─ integration/
   │  └─ e2e/
   └─ tools/
      └─ doctor/
```

`commands/` remains in the repo for compatibility, but the long-term direction is skills-first.

## Success Metrics

- All tests pass with 80%+ coverage
- No security vulnerabilities
- Code is readable and maintainable
- Performance is acceptable
- User requirements are met

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/K00-Kazu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
