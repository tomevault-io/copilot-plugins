## cli-continues

> - Session data is NEVER modified. Any PR that writes to `~/.claude/`, `~/.codex/`, `~/.copilot/`, `~/.gemini/`, `~/.factory/`, `~/.cursor/`, or `~/.local/share/opencode/` is a critical bug.

# continues — Code Review Standards

## Read-Only Semantics

- Session data is NEVER modified. Any PR that writes to `~/.claude/`, `~/.codex/`, `~/.copilot/`, `~/.gemini/`, `~/.factory/`, `~/.cursor/`, or `~/.local/share/opencode/` is a critical bug.
- Handoff files (`.continues-handoff.md`) write only to the project's own working directory — never to tool storage paths.

## ESM Conventions

- All local imports must use `.js` extensions, even for `.ts` source files: `import { foo } from './bar.js'`
- Never use `require()` — this is an ESM-only codebase (`"type": "module"` in `package.json`)
- Use `process.exitCode = N` instead of `process.exit(N)` — allows cleanup handlers to run

## Error Handling

- Use typed errors from `src/errors.ts` (`ParseError`, `SessionNotFoundError`, `ToolNotAvailableError`) for user-facing error paths — not bare `throw new Error()`
- Parsers must silently skip malformed session data with `catch {}` — never propagate parse errors to the caller

## Registry Completeness

- Every `SessionSource` member in `src/types/tool-names.ts` must have a registered `ToolAdapter` in `src/parsers/registry.ts`
- The completeness assertion at the bottom of `registry.ts` throws at module load if any adapter is missing — a missing entry crashes the CLI at startup
- Adding a tool to `TOOL_NAMES` without a registry entry is always a bug; changes to `SessionSource` require coordinated updates in the registry, fixtures, and tests

## Code Quality

- Biome handles all formatting and style — do not introduce ESLint, Prettier, or TSLint configs
- No synchronous file I/O (`readFileSync`, `writeFileSync`) in parser code — blocks the event loop when scanning large session directories
- Shared helpers `cleanSummary`, `extractRepoFromCwd`, and `homeDir` live in `src/utils/parser-helpers.ts` — do not duplicate them in individual parsers

## Performance

- Parsers run in parallel via `Promise.allSettled` in the session index builder — a slow parser delays only its own results
- The session index uses a 5-minute TTL cache at `~/.continues/sessions.jsonl` — cache bypasses should be explicitly justified

---
> Source: [yigitkonur/cli-continues](https://github.com/yigitkonur/cli-continues) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
