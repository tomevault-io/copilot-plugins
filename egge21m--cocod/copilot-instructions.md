## cocod

> This repository is a small Bun + TypeScript CLI/daemon.

# AGENTS.md

This repository is a small Bun + TypeScript CLI/daemon.
Agents should optimize for: minimal diffs, strict types, Bun-native APIs, and predictable CLI UX.

## Ground Rules (Repo Policy)

- Runtime: Bun (not Node). Prefer Bun APIs over Node/polyfills.
- Module system: ESM (`"type": "module"` in `package.json`).
- TypeScript is the linter: `tsc --noEmit` is the primary check.
- No Cursor/Copilot rule files were found (`.cursor/rules/**`, `.cursorrules`, `.github/copilot-instructions.md`).
- Also follow `CLAUDE.md` (Bun defaults, preferred APIs, testing conventions).

## Project Shape

- `src/index.ts`: entrypoint for the `cocod` binary (shebang: `#!/usr/bin/env bun`).
- `src/cli.ts` + `src/cli-shared.ts`: Commander-based CLI.
- `src/daemon.ts`: background daemon implemented with `Bun.serve()` on a UNIX socket.
- `src/routes.ts`: HTTP route handlers for the daemon (endpoints like /balance, /receive, /init, etc.).
- `src/utils/`:
  - `state.ts`: DaemonStateManager and wallet state logic
  - `wallet.ts`: Wallet initialization helpers
  - `crypto.ts`: Mnemonic encryption/decryption
  - `config.ts`: Configuration paths, env vars, and types
- IPC: CLI talks to daemon via `fetch()` with Bun's `RequestInit.unix` option.

Key paths/env:

- Socket: `COCOD_SOCKET` (default `~/.cocod/cocod.sock`).
- PID file: `COCOD_PID` (default `~/.cocod/cocod.pid`).
- Wallet config: `~/.cocod/config.json` (generated; do not commit).

## Commands

All commands run from repo root (`/home/egge/projects/cocod`).

### Install

```sh
bun install
```

### Run the CLI (foreground)

- Entrypoint:

```sh
bun src/index.ts --help
```

- Via npm-style script (use `--` to pass args):

```sh
bun run start -- --help
```

- Common commands:

```sh
bun src/index.ts balance
bun src/index.ts ping
bun src/index.ts mint list
```

### Start the daemon

The daemon can be started explicitly, but the CLI also auto-starts it when needed.

```sh
bun run daemon
```

### Build / bundle

There is no required build step (the CLI runs directly via Bun + TypeScript).

- Optional: produce a bundled artifact:

```sh
bun build src/index.ts --outdir dist --target bun
```

### Lint / typecheck

This repo currently uses TypeScript as the main lint gate.

```sh
bun run lint
```

If you need to run tsc directly:

```sh
bunx tsc --noEmit
```

### Tests

Bun's test runner is the expected choice.

- Run all tests:

```sh
bun test
```

- Run a single test file:

```sh
bun test path/to/file.test.ts
```

- Run a single test by name (recommended when adding tests):

```sh
bun test -t "ping returns pong"
```

Notes:

- Prefer test files named `*.test.ts` and colocated near the code they test.
- Use `import { test, expect } from "bun:test";`.

## Code Style

There is a formatter config at `.prettierrc`. Match existing style and avoid drive-by reformatting.

### Formatting

- Indentation: 2 spaces.
- Quotes: double quotes for strings.
- Semicolons: use them consistently (match surrounding file).
- Line length: keep lines reasonably short; wrap long function signatures.

### Imports

- Order:
  1. external packages
  2. blank line
  3. local relative imports
- Prefer `import type { ... }` for type-only imports.
- Prefer named imports; avoid default imports unless the package is default-first.
- Keep relative imports extensionless (match current code). Only include `.js` when required by ESM packages.

### Types and strictness

`tsconfig.json` enables strict TypeScript plus `noUncheckedIndexedAccess`.

- Avoid `any`. Use `unknown` at boundaries and narrow.
- Validate untrusted inputs (CLI args, request bodies, env vars).
- When indexing objects, handle `undefined` explicitly (e.g., `balance[mintUrl] || 0`).
- Prefer explicit return types on exported functions and non-trivial helpers.
- Use discriminated unions / literal types for protocol-like payloads.

### Naming

- Files: kebab-case for multiword modules (e.g., `cli-shared.ts`).
- Types/interfaces: `PascalCase`.
- Functions/variables: `camelCase`.
- Constants: `UPPER_SNAKE_CASE` for configuration-like values.
- CLI commands: lowercase; nouns/verbs consistent with existing Commander usage.

### Error handling

General:

- Only `process.exit()` from true CLI entrypoints.
- Prefer throwing `Error` (or subclasses) from library-ish functions.
- In `catch`, treat the error as `unknown`; derive a safe message:
  - `error instanceof Error ? error.message : String(error)`.

Daemon (`src/daemon.ts`):

- Return JSON with either `{ output: string }` or `{ error: string }`.
- Use proper HTTP status codes for failures (e.g., 400 for bad input, 404 unknown endpoint, 500 unexpected).
- Do not swallow errors silently; if you intentionally suppress errors (e.g., delete stale files), add a short comment.

CLI (`src/cli-shared.ts`):

- Print user-facing errors to stderr (`console.error`).
- Exit with code 1 for expected failures.
- For daemon connectivity issues, prefer actionable messages (socket path, how to start daemon).

### Bun-specific guidance (from `CLAUDE.md`)

- Use `bun <file>` / `bun run <script>` (not `node`, `ts-node`, `npm`).
- Use `bun test` (not jest/vitest).
- Use `Bun.serve()` routes/websocket support (not express).
- Prefer `Bun.file` for file IO; Bun loads `.env` automatically (avoid `dotenv`).
- Prefer Bun-native DB/network libs where applicable:
  - `bun:sqlite` for SQLite (avoid `better-sqlite3`)
  - `Bun.sql` for Postgres (avoid `pg`)
  - `Bun.redis` for Redis (avoid `ioredis`)
  - built-in `WebSocket` (avoid `ws`)

## Editing Expectations for Agents

- Keep diffs surgical; do not reformat unrelated code.
- Preserve CLI UX and backward compatibility of command names/flags unless explicitly requested.
- Avoid committing/generated artifacts (e.g., `coco.db`, sockets, pid files, `.env`).
- When adding new commands/routes:
  - update Commander wiring in `src/cli.ts`
  - add/update the route handler in `src/routes.ts` (and ensure it returns the `{ output | error }` shape)
  - keep the CLI/daemon contract explicit (method, path, request/response types).
  - update `docs/daemon-api.json` to keep the route contract in sync.

## Agent Playbook

Use these repeatable recipes to keep changes predictable.

### Add or update a CLI command

1. Update command wiring in `src/cli.ts`.
2. Add/update daemon route in `src/routes.ts`.
3. Keep route responses to `{ output: ... }` on success and `{ error: string }` on failure.
4. Return explicit status codes:
   - 400 invalid input
   - 401 auth/passphrase failure
   - 403 locked wallet for unlocked-only endpoints
   - 404 unknown endpoint
   - 409 state conflict
   - 500 unexpected runtime failure
5. Update docs:
   - `README.md` command tables/examples
   - `docs/daemon-api.json` endpoint contract
6. Add/adjust tests and run checks (`bun run lint`, `bun test`).

### Validate request bodies

- Treat request bodies as untrusted.
- Validate required fields before calling wallet methods.
- Return 400 with actionable messages for malformed JSON, missing fields, wrong types, or invalid ranges.

### Keep docs and runtime aligned

- If you change command names, env var behavior, or defaults, update docs in the same patch.
- Prefer one source of truth for path constants (`src/utils/config.ts`).
- Do not leave placeholders in user-facing docs.

## Quick Debugging Checklist

- CLI can't connect: verify `COCOD_SOCKET` matches daemon and the socket exists.
- Daemon won't start: check for stale `~/.cocod/cocod.sock` and `~/.cocod/cocod.pid`.
- Type errors: run `bun run lint` and fix strictness issues (especially `undefined` from indexing).

---
> Source: [Egge21M/cocod](https://github.com/Egge21M/cocod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
