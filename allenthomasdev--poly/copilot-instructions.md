## poly

> - **Test all:** `npm test` | **Single test:** `npx vitest run test/foo.test.ts` | **Watch:** `npm run test:watch`

# AGENTS.md

## Commands
- **Test all:** `npm test` | **Single test:** `npx vitest run test/foo.test.ts` | **Watch:** `npm run test:watch`
- **Lint:** `biome check src/ test/` | **Fix:** `biome check --write .` | **Typecheck:** `tsc --noEmit`
- **Build:** `npm run build` | **Dev:** `npm run dev`

## Architecture
Poly is a tmux pane orchestration tool: CLI (`src/cli/`) → Hono HTTP server (`src/server/`, default port 4096) → extension system (`src/extensions/`). Extensions register actions and event handlers via `ExtensionAPI`; the runner executes actions and emits events. Core actions (pane CRUD, meta, session) live in `src/extensions/core-actions.ts`. `src/tmux.ts` wraps all tmux subprocess calls. User extensions go in `~/.poly/extensions/`. Tests in `test/` use a shared `harness.ts` (server spawning, tmux socket isolation, SSE frame reading, state reset).

## Code Style
- **Runtime:** Node.js (ESM only, `"type": "module"`). TypeScript with strict mode + `noUncheckedIndexedAccess`.
- **Formatting:** Biome — 2-space indent, 120 line width, double quotes, semicolons only as-needed.
- **Imports:** Use `node:` prefix for Node builtins. Use `verbatimModuleSyntax` (use `import type` for type-only imports).
- **Validation:** Zod for runtime validation (request bodies, config). Prefer `z.object()` schemas.
- **HTTP:** Hono framework. Routes in `src/server/routes/`. JSON responses.
- **Logging:** Use `log()` from `src/constants.ts` (structured JSON to stderr, silenced in tests via `NODE_ENV=test`).
- **Error handling:** Throw errors with descriptive messages; top-level catches in CLI and server entry. Fire-and-forget for non-critical event handlers.
- **Naming:** camelCase for variables/functions, PascalCase for types. Files are kebab-case. Test files: `foo.test.ts`.
- **Testing:** `vitest` with `describe`/`test`/`expect`. Use `harness.ts` helpers (`resetState`, `makeTmux`, `startServer`, `request`, `readSSEFrames`). Call `resetState()` in `beforeEach`.
- **Dependencies:** `yargs` for CLI, `hono` for HTTP, `zod` for validation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AllenThomasDev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AllenThomasDev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
