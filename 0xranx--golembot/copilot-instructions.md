## testing-conventions

> Testing conventions — must reference when modifying src/ code or tests


# Testing Conventions

## Feature Changes Must Be Verified (Most Important Rule)

**Any substantive change to source files under `src/` must:**

1. **Add dedicated tests for the new functionality** — passing existing tests is not enough. Each new feature/field/parameter needs corresponding test cases; these must fail when the feature is removed.
2. Evaluate whether **e2e tests** need updates — if the change affects interaction with the real Agent (e.g. params, output parsing, event types), add new assertions in e2e.
3. **Run e2e once** (`pnpm run e2e`) to ensure real scenarios still work — even if the change seems small.

### ⚠️ Anti-Pattern Warning

**"Changed code + all old tests pass = done" — that is wrong.** Old tests only verify old behavior. New functionality without dedicated tests is effectively unverified.

Typical mistakes:
- Added `apiKey` to `InvokeOpts` → but no test verifies `apiKey` is actually passed to the engine
- Added `durationMs` to `done` event → but no test for with/without `duration_ms` scenarios
- Modified `parseStreamLine` → but e2e has no assertion for the new fields returned by the real Agent

Correct approach:
1. Unit tests: mock engine captures new params; verify end-to-end propagation of new fields from source to consumer
2. e2e assertions: verify new fields have values in real conversations (e.g. `durationMs > 0`)
3. Do not consider it done until tests have been run

## Unit Tests (vitest)

Location: `src/__tests__/<module>.test.ts`

### Mock Engine Pattern

Unit tests do not call the real Agent (Cursor / Claude Code). Replace with a mock engine:

```typescript
async function* mockInvoke(prompt, opts): AsyncIterable<StreamEvent> {
  yield { type: 'text', content: 'mock response' };
  yield { type: 'done', sessionId: 'mock-session-123' };
}
```

Inject via `vi.mock('../engine.js', ...)`, returning mock engine from `createEngine`.

### Key Test Scenarios

- **session.test.ts**: Multi-key routing, Phase 1 format migration, concurrency safety
- **index.test.ts**: sessionKey routing, KeyedMutex concurrency isolation, resetSession
- **server.test.ts**: All HTTP endpoints, SSE streaming, Bearer auth, CORS, input validation
- **engine.test.ts**: stream-json parsing (parseStreamLine + parseClaudeStreamLine), ANSI stripping, injectClaudeSkills, createEngine factory
- **workspace.test.ts**: golem.yaml read/write, skills scanning, AGENTS.md generation

### Running

```bash
pnpm run test         # Single run
pnpm run test:watch   # Watch mode
```

## End-to-End Tests

### Three e2e Test Files

| File | Command | Scenario | Prerequisites |
|------|---------|----------|---------------|
| `examples/e2e-test.ts` | `pnpm run e2e` | Cursor engine full feature verification (IM / CI / Pipeline / HTTP) | Cursor Agent CLI |
| `examples/e2e-headless.ts` | `pnpm run e2e:headless` | Cursor headless deployment verification (apiKey / HTTP / service restart) | Cursor Agent CLI + `CURSOR_API_KEY` |
| `examples/e2e-claude-code.ts` | `pnpm run e2e:claude-code` | Claude Code engine verification (chat / multi-turn / session / HTTP / costUsd) | Claude Code CLI + `claude auth` or `ANTHROPIC_API_KEY` |

### .env Auto-Loading

All three e2e scripts auto-load env vars from the project root `.env` file (without overwriting existing vars). Sensitive values (`CURSOR_API_KEY`, `ANTHROPIC_API_KEY`) go in `.env`; no need to pass them manually each time.

`.env` is in `.gitignore`; `.env.example` provides a template without secrets.

### Design Principles

1. **Use natural language prompts** — Do not specify Skill names, script paths, or filenames in prompts. Let the Agent discover and use Skills on its own.
2. **Loosen assertions** — Agent may choose different filenames or output directly in the reply without writing a file. Assertions should tolerate this flexibility.
3. **Isolated temp directories** — Each test scenario uses `mkdtemp()` for an isolated directory; cleaned up after the test.
4. **Check concrete data** — Do not just check "there is output"; verify the output contains expected values or keywords.

### e2e-test.ts Structure

- PART 1: Core engine capabilities (chat, durationMs validation, file I/O, Skill hot-swap, script invocation, persistent memory)
- PART 2: Embedded scenarios (IM bot, CI/CD code review, data pipeline)
- PART 3: Multi-instance isolation verification
- PART 4: HTTP service (startup, health, auth, multi-user session, reset)

### e2e-headless.ts Structure

- PART 1: Initial deployment (apiKey create assistant + Skill config)
- PART 2: apiKey direct chat (auth chain + durationMs + session persistence)
- PART 3: HTTP service production deployment (health, auth, ops log analysis, monitoring data analysis, multi-tenant isolation, session reset)
- PART 4: Service restart (session persistence across restarts + multi-turn chat after restart)

### Notes

- Cursor e2e requires Cursor Agent CLI available (`~/.local/bin/agent`)
- Claude Code e2e requires Claude Code CLI available (`~/.local/bin/claude`) + auth
- e2e-test.ts takes ~5–7 minutes per run; e2e-headless.ts ~2–3 min; e2e-claude-code.ts ~1–2 min
- On failure, process.exit(1), with clear markers for which assertions failed
- e2e-headless.ts auto-skips when `CURSOR_API_KEY` is missing (exit 0)
- e2e-claude-code.ts auto-skips when Claude Code CLI is unavailable or not authenticated and no `ANTHROPIC_API_KEY` (exit 0)

---
> Source: [0xranx/golembot](https://github.com/0xranx/golembot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
