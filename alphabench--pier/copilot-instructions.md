## pier

> pier is a TypeScript + Bun coding-agent CLI. It runs an interactive REPL (or headless

# pier — AGENTS.md

pier is a TypeScript + Bun coding-agent CLI. It runs an interactive REPL (or headless
`exec`/`serve`), drives an agentic tool loop against models served through the Pier bridge
(with BYOK support for Sarvam AI and OpenRouter), and ships as a single compiled binary.

## Quick reference

| Command | What it does |
|---------|-------------|
| `bun test` | Run all tests (bun:test) |
| `bun run dev` | Run without compiling (`bun src/entry.ts`) |
| `bun run build` | Compile → `dist/pier` |
| `bun run typecheck` | `tsc --noEmit` |
| `bun run lint` | `biome check src scripts` |
| `bun run format` | `biome format --write src scripts test` |
| `bun run verify` | typecheck + lint + format:check + test |

Run a single test file first: `bun test test/<file>.test.ts`, then the full suite.

## Project structure

```
src/
  entry.ts              — bin entry shim (loads MACRO global, boots SRT, calls runCli)
  cli.ts                — Commander CLI multitool (repl, exec, serve, login, keys, …)
  ink.ts                — thin facade over the ink renderer + design-system components
  _macros/              — build-time constants (BUILD, feature flags, MACRO global)
  protocol/             — Responses API wire types (items.ts), model catalog (models.ts),
                          approval/sandbox types, bundled models_catalog.json
  bridge/               — model client: SSE frame parser, event validator, POST /v1/responses,
                          GET /v1/models, tips, subscription
  auth/                 — device pairing (OAuth device-code flow) + token storage
  engine/               — the turn loop: callModel, translate-in/out, turn.ts (agentic loop),
                          session.ts (mutable state), controller.ts (headless session driver),
                          exec.ts (headless path), compact/microcompact, retry, fallback,
                          checkpoint, planFile, ultracode, backgroundTasks, monitors
  tools/native/         — native tool handlers (Read, Write, Edit, Grep, Bash, Task, …)
                          + registry.ts (filters tools by permission mode)
  safety/               — permission engine, sandbox adapter, trust model, auto-classify
  hooks/                — lifecycle hooks (PreToolUse, PostToolUse, SessionStart, Stop, …)
  agents/               — subagent loading + execution (built-in + disk-defined)
  skills/               — SKILL.md-based skill bundles
  mcp/                  — MCP client (stdio + streamable-http)
  commands/             — slash commands + custom commands from $PIER_HOME/commands/*.md
  context/              — project memory (CLAUDE.md / AGENTS.md discovery + @import)
  providers/            — BYOK provider registry (Sarvam AI, OpenRouter), chat completions
                          client, credential management, model resolution
  server/               — `pier serve` headless session server (stdio + register-out WS)
  runner/               — runner adapter for the bridge's runner pool
  threads/              — thread persistence (local JSON store + bridge sync)
  tui/                  — REPL: composer, transcript cells, dialogs, onboarding, model picker
  terminal/             — ANSI rendering, hyperlinks, clipboard, titled panes, string width
  components/           — design-system (ThemedBox, ThemedText, ThemeProvider, color)
  config/               — build channel, PIER_HOME resolution, internal-access gate
  utils/                — config, fs, log, sanitize, schedule, theme, env, debug, …
  types/                — message.ts (internal message union the whole engine/UI speaks)
packages/
  protocol/             — @pier/serve-protocol: Zod schemas for the serve NDJSON protocol
test/                   — ~100 test files, one per module/feature (bun:test, no describe/it)
scripts/                — build, release, install, model sync, vendor-rg
vendor/rg/              — bundled ripgrep binaries (darwin, linux, windows, arm64/x64)
```

## Key architecture

- **Entry flow**: `entry.ts` → `cli.ts` → `launchRepl()` (TUI) or `runExec()` (headless) or
  `runServe()` (embedding server). The REPL creates a `Session` + `SessionController` and
  renders a `Repl` component through ink.
- **Turn loop** (`engine/turn.ts`): streams a model call via `callModel`, collects tool_use
  blocks, gates each through the approval policy (permission mode + safety layer), runs the
  tool, appends tool_result, and loops until `stop_reason !== 'tool_use'`.
- **Session state** (`engine/session.ts`): a mutable `Session` object threads through the
  whole turn loop — cwd, model, instructions, history, permission mode, sandbox policy,
  token usage, checkpoints, todos, and the tool allowlist.
- **Permission modes**: `default` (prompt on writes/exec), `plan` (read-only + ExitPlanMode),
  `acceptEdits` (auto-approve edits), `auto` (classifier decides, fails safe to prompt),
  `bypassPermissions` (everything auto-approved). Cycled with Shift-Tab in the REPL.
- **OS sandboxing**: Bash runs inside a real OS sandbox (seatbelt on macOS, bubblewrap on
  Linux) sized to the active permission mode. The sandbox blocks outbound network by default;
  `needs_network: true` requests it per command.
- **Project memory** (`src/context/projectMemory.ts`): discovers `CLAUDE.md` / `AGENTS.md`
  from `$PIER_HOME` and from the repo root down to cwd, resolves `@import` directives
  recursively, and prepends the result to session instructions.
- **Model protocol**: the CLI speaks only the OpenAI Responses API. The Pier Go bridge
  translates to Chat Completions. Wire types are in `src/protocol/items.ts`; the internal
  message model is content-block-based (`src/types/message.ts`); translators in
  `engine/translate-in.ts` / `engine/translate-out.ts` convert between them.
- **BYOK providers**: Sarvam AI and OpenRouter are registered in `src/providers/registry.ts`.
  Each has a `ProviderDef` with base URL, key env var, validation probe, and model list.
  Keys are stored in `$PIER_HOME/credentials.json` (0600) or set via env vars.

## Conventions

- **TypeScript + Bun**: ESNext modules, `bun:test` for testing, `bun` as runtime/package
  manager. No `describe`/`it` — use `test()` from `bun:test` directly.
- **Formatting**: Biome with 2-space indent, 100-char line width, single quotes,
  `asNeeded` semicolons, trailing commas. Run `bun run format` before committing.
- **Imports**: use `.js` extensions in import paths (Bun convention for ESM). Path aliases
  via `src/*` → `./src/*` in tsconfig.
- **No `any` by default**: `noExplicitAny` is off in Biome, but prefer proper types.
  `noNonNullAssertion` is off too — use `!` sparingly.
- **Error handling**: best-effort for non-critical paths (checkpoints, persistence, MCP,
  hooks). A failure in these never wedges the session. Use try/catch with silent fallback
  for best-effort operations.
- **Testing**: one test file per module/feature in `test/`. Tests use `mock.module()` from
  `bun:test` for module-level mocking. Prefer testing through the public API of a module
  rather than internal details.
- **Git**: branch off `dev`, rebase onto `dev` before PRs, squash-merge into `dev`.
  Releases fast-forward `main` to a tested `dev` commit.
- **Comments**: JSDoc on every module (`/** … */`). Explain *why*, not *what*. Use
  `GOTCHA:` / `NOTE:` markers for non-obvious constraints.
- **Feature flags**: use `feature('FLAG_NAME')` from `src/_macros/features.ts`. Flags
  default off; enable in the `ENABLED` set. Override at runtime with `PIER_FEATURE_<FLAG>=1`.
- **Build-time constants**: `MACRO.VERSION`, `MACRO.BUILD_TIME`, etc. are injected at
  compile time via `PIER_BUILD_*` env vars. In dev they resolve to `0.0.0-dev`.

## Key files to know

- `src/engine/turn.ts` — the agentic turn loop (the heart of the system)
- `src/engine/session.ts` — Session type + createSession
- `src/engine/controller.ts` — headless SessionController (used by REPL and serve)
- `src/engine/callModel.ts` — the model-call seam
- `src/tools/native/registry.ts` — tool registration + permission-mode filtering
- `src/safety/safety.ts` — the permission gate (assessToolCall)
- `src/protocol/items.ts` — Responses API wire types
- `src/types/message.ts` — internal message union
- `src/context/projectMemory.ts` — CLAUDE.md / AGENTS.md discovery
- `src/tui/Repl.tsx` — the interactive REPL component
- `src/tui/launch.tsx` — REPL launcher (session construction + ink render)
- `src/cli.ts` — all CLI commands
- `src/engine/exec.ts` — headless exec path
- `src/server/serve.ts` — headless serve path
- `src/hooks/hooks.ts` — lifecycle hooks system
- `src/engine/init.ts` — session initialization (MCP, skills, hooks)
- `packages/protocol/src/index.ts` — serve protocol Zod schemas

---
> Source: [alphabench/pier](https://github.com/alphabench/pier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
