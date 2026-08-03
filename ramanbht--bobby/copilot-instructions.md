## bobby

> Guidance for Claude Code (and other agents) working in this repo. Keep it short;

# CLAUDE.md

Guidance for Claude Code (and other agents) working in this repo. Keep it short;
update it when an architectural rule changes.

## What Bobby is

A local-first, multi-harness chat dashboard. It drives the harness CLIs you
already run (`claude`, `hermes`, `pi`), normalizes their output into one event
stream, persists every chat to SQLite, and can distill chats into an Obsidian
vault. Browser tab (`pnpm dev`) or native desktop app (`pnpm desktop`).

## Layout

```
packages/
  shared/   Types + wire protocol shared by server & web (the contract)
  server/   Fastify + WebSocket backend, SQLite store, harness adapters, memory pipeline
  web/      Vite + React chat UI
  desktop/  Electron shell (boots the server in-process)
```

## Architecture in one breath

```
React UI ──WS──> Fastify server ──> Adapter (per harness) ──spawn──> claude/hermes/pi CLI
                      │  persists HarnessEvent stream to SQLite (canonical)
                      └─ Distiller ──> Obsidian vault
```

- Each **adapter** (`packages/server/src/adapters/`) is the *only* code that knows
  a harness's CLI quirks. It spawns the CLI and yields a normalized
  `HarnessEvent` stream (`session` / `text-delta` / `text` / `thinking-delta` /
  `tool-use` / `tool-result` / `done` / `error`) defined in `@bobby/shared`.
- The **server** persists that stream to SQLite and forwards it to the browser as
  `ServerFrame`s over `/ws`. The UI only ever speaks this one normalized dialect.
- A **turn** spawns a fresh harness process that exits when the turn ends. Idle
  chats hold no process; N concurrent active turns = N transient processes.

## Non-negotiable principles

- **SQLite is the source of truth** for chat history. Harness-native sessions
  (Claude `-r`, pi `--session`) are a resume optimization, never the canonical
  record. When harness continuity breaks (e.g. switching harness mid-chat),
  clear the native session id and replay Bobby's stored history.
- **Adapters are the only harness-aware code.** Don't leak CLI flags or quirks
  into the server, the UI, or shared types.
- **One shared data dir for every launch method.** The dev/prod server and the
  desktop app all read/write the same DB (`config.ts` → `appDataDir()`:
  `~/Library/Application Support/Bobby` on macOS; the desktop shell pins
  `app.setName("Bobby")` so its userData resolves there too). Override with
  `BOBBY_DB` / `BOBBY_WORKDIR` (the test harness does this to stay isolated).
  Don't reintroduce a per-package `data/` dir — that silently splits history.

## Adding a harness

1. Implement the adapter in `packages/server/src/adapters/<id>.ts` (normalize CLI
   output to `HarnessEvent`).
2. Register it in `packages/server/src/adapters/index.ts`.
3. Add its id to `HarnessId` / `HARNESSES` in `packages/shared/src/index.ts`.
4. Add parser unit tests (see "Testing" below). Persistence, streaming, and UI
   are harness-agnostic and pick it up automatically.

## Conventions

- TypeScript, ES modules, `strict` on. Import sibling server modules with the
  `.js` extension (e.g. `import { config } from "./config.js"`).
- Keep cross-process types in `@bobby/shared` — nothing else crosses the wire.

## Footguns (read before editing)

- **Edited `packages/shared/src`? Rebuild it.** The server and web import the
  *built* `@bobby/shared` (`dist/`), so a `tsc`/typecheck against stale types
  will fail or lie. Run `pnpm --filter @bobby/shared build` (or `pnpm typecheck`,
  which builds shared first via the root `test`/`dev` scripts) after changing
  shared types.
- **SQLite column-order drift.** Older DBs added some columns via migration
  (`ensureColumn`), so column order can differ between databases. Always use
  **explicit column names** in INSERT/SELECT — never `SELECT *` into a positional
  insert.
- **Distillation vault** resolves via `db.effectiveObsidianVault()` = Settings
  value first, then the `OBSIDIAN_VAULT` env var, else off. Use that helper; don't
  read `config.obsidianVault` directly in new code.
- **better-sqlite3 is a native module.** `pnpm desktop` runs the Electron app
  against system-Node's build of it; if you ever see an ABI mismatch on boot, run
  `pnpm install` (or `pnpm rebuild better-sqlite3`) to recompile it.

## Testing — the gate

```bash
pnpm e2e        # typecheck + vitest + build + REST/WS smoke; exits non-zero on any failure
pnpm test       # just the vitest suite (offline; builds shared first)
pnpm typecheck  # all packages
```

`pnpm e2e` must pass before committing/pushing. Test patterns:

- **Parsers / pure functions** — unit-test harness-output mappers directly
  (`packages/server/test/parsers.test.ts`). New adapter ⇒ add parser tests.
- **HTTP API** — Fastify `app.inject` end-to-end (`test/*api*`).
- **Orchestration** — use the mock-adapter pattern in
  `test/orchestration.test.ts` for anything in `turn.ts` that streams or mutates
  messages (runTurn, runPlan, executePlan/continuePlan, stopChat, editAndRerun).

## Git

Don't commit unless asked. `pnpm e2e` green is a precondition for any commit/push.
Never skip hooks or force-push `main`.

---
> Source: [ramanbht/bobby](https://github.com/ramanbht/bobby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
