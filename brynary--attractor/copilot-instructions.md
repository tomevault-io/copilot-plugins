## attractor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
bun install

# Run tests (per workspace)
cd unified-llm && bun test
cd coding-agent && bun test
cd attractor && bun test

# Run a single test file
bun test unified-llm/tests/utils/retry.test.ts

# Run root-level E2E tests
bun test tests/

# Type check (per workspace)
cd unified-llm && bun run typecheck
cd coding-agent && bun run typecheck
cd attractor && bun run typecheck

# Start HTTP server
bun run attractor/bin/attractor-server.ts
```

## Architecture

Bun/TypeScript monorepo with three workspace packages. Dependency direction is strictly bottom-up:

```
unified-llm  →  coding-agent  →  attractor
(LLM client)    (agentic loop)   (DOT pipeline engine)
```

**unified-llm** -- Unified LLM client across Anthropic, OpenAI, Gemini, and OpenAI-compatible providers. Four layers: types (L1) → utils (L2) → client (L3) → high-level API (L4). Each provider has an adapter with request/response/stream translators in `src/providers/`. Only external dependency: `ajv`.

**coding-agent** -- Turns an LLM into a code-editing agent. Core is `Session` (`src/session/session.ts`) which runs the agentic loop with tool execution, history management, loop detection, and subagent orchestration. Provider-aligned profiles in `src/profiles/` configure per-provider system prompts and tool shapes. Tools (read_file, write_file, edit_file, shell, grep, glob) in `src/tools/`. `ExecutionEnvironment` interface (`src/env/`) abstracts where tools run.

**attractor** -- DOT-based pipeline orchestration. Parses Graphviz DOT files (`src/parser/`) into graphs, then `PipelineRunner` (`src/engine/runner.ts`) traverses them. 10 node handlers in `src/handlers/` (start, exit, codergen, conditional, wait-human, parallel, fan-in, tool, manager-loop, sub-pipeline). Backends in `src/backends/` connect to coding agents. Interviewers in `src/interviewer/` handle human-in-the-loop approval patterns.

## Spec-Driven Development

Three NLSpec documents define the system and are the source of truth:
- `docs/specs/attractor-spec.md` -- Pipeline engine
- `docs/specs/unified-llm-spec.md` -- LLM client
- `docs/specs/coding-agent-loop-spec.md` -- Agentic loop

## Key Patterns

- **Interface-first**: All major components defined as interfaces (`Handler`, `CodergenBackend`, `Interviewer`, `ProviderAdapter`, `ExecutionEnvironment`, `ProviderProfile`)
- **Discriminated unions**: Types use `kind` discriminants extensively (e.g., `ContentPart`, `AttributeValue`)
- **Adapter + translator**: Each LLM provider has an adapter that delegates to separate request-translator, response-translator, and stream-translator modules
- **Registry pattern**: `HandlerRegistry` maps node shapes to handlers; `ToolRegistry` maps names to tool definitions
- **Factory functions**: `createHandlerRegistry()`, `createAnthropicProfile()`, `createReadFileTool()`, etc.
- **Async generators**: Event streaming uses `AsyncGenerator` in both `PipelineEventEmitter` and `Session.events()`
- **Imports use `.js` extensions** (Bun ESM convention)

## Testing

- Bun's built-in test runner (`bun:test`) with `describe`/`test`/`expect`
- Test doubles are stubs/fakes, not mocks: `StubAdapter` (`unified-llm/tests/stubs/`), `StubExecutionEnvironment` (`coding-agent/tests/stubs/`)
- Test helpers (e.g., `makeNode()`, `makeGraph()`, `createTestSession()`) are local functions at the top of each test file
- Cross-workspace imports in tests use package names: `import { StubAdapter } from "unified-llm/tests/stubs/stub-adapter.js"`

### SessionBackend in tests

- Create a **fresh `Client` + `SessionBackend` per test**, not a shared instance in `beforeAll`. Shared backends hang in bun's test runner. See `attractor-real-llm.test.ts` for the working pattern.
- **Constrain prompts** when using a real LLM via `SessionBackend`: add "Do not use tools" and length constraints like "in 1-2 sentences". Without this, the coding-agent `Session` enters tool-use loops (file creation, shell commands) that hang or run for minutes. Only scenario 5-style tests that intentionally test file creation should allow tool use.
- When loading `.env` in tests, **don't overwrite env vars already set** (`!(key in process.env)`). This allows `ANTHROPIC_API_KEY= bun test` to force stub mode even when `.env` has a key.

## TypeScript

- `strict: true` with `noUncheckedIndexedAccess: true`
- `moduleResolution: "bundler"`, `target`/`module`: `ESNext`
- No build step -- Bun runs TypeScript directly; `tsc --noEmit` is for type checking only

## Commits

Conventional commits: `feat(unified-llm):`, `fix(coding-agent):`, `docs:`, `chore:`

---
> Source: [brynary/attractor](https://github.com/brynary/attractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
