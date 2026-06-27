## elliot

> Elliot is a platform that turns **existing products into agentic products**.

# Elliot — CLAUDE.md

## Project Mission

Elliot is a platform that turns **existing products into agentic products**.

The target user is a product engineer who has a working API or database today and wants AI agents to interact with it natively — with minimum tokens, clean error recovery, and full observability. Elliot provides the tools to design, validate, deploy, and observe agent-ready tool sets built around any API or DB.

The five principles that drive every technical decision:
1. Tool descriptions are contracts — verb-first, unambiguous, typed
2. Results are sized for context windows — not raw table dumps
3. Errors are actionable — agents must know what to do next
4. Every agent session is observable — token cost, latency, errors, all visible
5. The platform itself is agentic — agents can build connectors through Elliot

---

## Before Every Push — Mandatory Checks

```bash
uv run ruff check .
uv run ruff format --check .
uv run mypy packages/core/src packages/mcp-plugin/src packages/connector-runtime/src
uv run pytest --tb=short
pnpm --filter @elliot/studio run typecheck
pnpm --filter @elliot/studio test --run
```

All six must pass. Fix the issue, re-run from the top, then push.

---

## Branch Strategy

```
main
└── epic/01-monorepo-setup
    ├── task/001-root-workspace
    ├── task/002-python-tooling
    └── task/004-package-stubs
└── epic/02-core-library
    ├── task/005-core-type-definitions
    └── ...
```

### Rules
1. Epic branch from `main`. Task branch from its epic branch.
2. Merge task → epic (no-ff). Merge epic → main only when all tasks pass.
3. Never commit directly to `main`.
4. Rename task file to `complete-NNN-name.md` on epic branch after merge.

---

## Task Completion Protocol

1. Read the task file in full before writing code.
2. Implement only what the task specifies — no extra features.
3. Write unit tests as part of the same commit.
4. Run the full check suite (above). Fix failures.
5. Push task branch. Merge into epic.
6. Rename task file: `001-root-workspace.md` → `complete-001-root-workspace.md`.
7. Commit the rename on the epic branch.

---

## Logging — Required Everywhere

Every Python module that does meaningful work gets a module-level logger:

```python
import structlog
log = structlog.get_logger(__name__)
```

Log at every significant boundary: service start, tool call start/complete/error, connector load, session open/close, secrets resolved, cache hit/miss.

**Never log:** secret values, API keys, raw query results (may contain PII).

TypeScript — prefix every `console.*` with `[module-name]` for DevTools filtering.

---

## Error Handling — Required Everywhere

- No bare `except Exception: pass` or silent failures.
- All Python errors inherit from `ElliotError` (task 052).
- Every MCP tool handler has a top-level `try/except` returning `to_mcp_error_content(exc)`.
- Stack traces go to the log (`exc_info=True`). Agents only see structured `{code, message}`.
- Error response shape: `{"error": {"code": "VALIDATION_*", "message": "...", "details": {...}}}`

---

## Code Standards

### Python
- All public functions have type annotations.
- Pydantic models for all data crossing a service boundary.
- `structlog` everywhere — never `print()` or bare `logging`.
- Secrets never in logs, never hardcoded — use `{{ env:VAR }}` in connector files.

### TypeScript
- `"strict": true` in tsconfig.
- No `any` — use `unknown` and narrow.
- All fetch calls through `src/client/http.ts` (API key injection).

### Tests
- Unit tests live in `tests/` at the package root.
- Integration tests marked `@pytest.mark.integration`.
- Coverage gates: `elliot-core` ≥ 95%, services ≥ 85%, studio ≥ 70%.
- Every error path must have a test.

### Commits
- Format: `type(scope): short description`
- Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`

---

## Project Structure

```
elliot/
├── packages/
│   ├── core/                  elliot-core          Python 3.13
│   ├── mcp-plugin/            elliot-mcp-plugin    :3000
│   ├── connector-runtime/     elliot-connector-runtime  :3001
│   └── studio/                elliot-studio        React 19 :5173
├── connectors/            user connector.json files
├── Procfile               honcho start
├── docker-compose.yml     production
└── .env.example           all env vars
```

## Key Ports

| Service | Port |
|---|---|
| `elliot-mcp-plugin` | 3000 |
| `elliot-connector-runtime` | 3001 |
| `elliot-studio` | 5173 |

---
> Source: [EliBarak12/Elliot](https://github.com/EliBarak12/Elliot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
