## pi-lsp-client

> Conventions for human contributors and AI agents working on this repository.

# Repository Conventions

Conventions for human contributors and AI agents working on this repository.

## Style

- Terse technical prose. No emojis in commits, issues, PR comments, or code.
- TypeScript strict mode. No `any`, no `unknown` casts where avoidable, no
  `@ts-ignore`, no `@ts-expect-error`, no enums.
- ESM modules with `.js` suffix in import paths (Node16 resolution).
- Tabs for indentation. Double quotes for strings (matches biome config).
- Tests use vitest with `#given .. #when .. #then` description style or plain
  `// given / // when / // then` body comments. No Arrange/Act/Assert labels.

## Commands

- `npm install` — install dependencies (peer + dev). Run after clone.
- `npm test` — run vitest test suite once.
- `npm run typecheck` — strict TypeScript check (no emit).
- `npm run check` — type check + biome.
- `pi -e ./src/index.ts` — load the extension into a local pi session for
  manual smoke testing.

## Constraints

- No Bun APIs. Runtime is Node only. The transport layer uses
  `node:child_process.spawn` and native Node streams.
- LSP server processes are managed exclusively by `LspManager`. Tools acquire
  via `withLspClient(...)` and the wrapper handles refCount + retry.
- `lsp_rename` runs with `executionMode: "sequential"` because it mutates
  files via a workspace edit spanning arbitrary paths.
- No raw `SIGINT`/`SIGTERM` handlers. Cleanup happens via
  `pi.on("session_shutdown")` plus an idempotent `process.once("exit")`
  fallback that returns a disposer.
- TUI rendering reads from typed `details` returned by tool `execute`.
  Renderers never parse formatter strings to recover details.
- Server installation is auditable: `/lsp install <id>` runs documented
  recipes from `AUTO_INSTALLABLE_SERVERS`. Any other id surfaces the
  manual install hint instead.

## Don'ts

- No `git add -A` or `git add .`. Stage only the files you changed.
- No `git commit --no-verify`. No force pushes. No history rewriting on
  shared branches.
- No new dependency on omo source paths. The package is standalone.
- No new dependency on pi-coding-agent internal modules outside the
  documented public extension API in `@mariozechner/pi-coding-agent`.
- Don't bypass the manager's refCount accounting. Always acquire via
  `withLspClient` (or `manager.getClient` + `releaseClient` paired in a
  `finally`) so idle reaping and shutdown work correctly.

---
> Source: [code-yeongyu/pi-lsp-client](https://github.com/code-yeongyu/pi-lsp-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
