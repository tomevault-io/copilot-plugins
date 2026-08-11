## ingit

> Always use **bun**, never npm.

# AGENTS.md

## Commands

Always use **bun**, never npm.

```sh
bun install               # install deps
bun dev                   # server (http://127.0.0.1:8488) + vite dev client
bun run dev:client        # client only
bun run typecheck         # tsc --build (project references) — must pass before shipping
bun test                  # run all tests
bun test packages/git-core/src/repo-session.test.ts   # single test file
bun test -t "name"        # single test by name
bun run build             # build all workspaces
bun run cli:release       # build standalone `ingit` binary (bun build --compile)
```

Tests live next to code as `*.test.ts` (currently mostly in `packages/git-core`).

## Architecture

Bun workspaces monorepo — a browser git GUI (vibecoded ungit alternative):

- `packages/rpc-contract` — zod-typed oRPC contract shared by client and server. **Single source of truth for the API**: to add an endpoint, define it in `src/contract.ts`, implement it in `apps/server/src/rpc-router.ts`, then call it type-safe from the client (`apps/client/src/api.ts`).
- `packages/git-core` — spawns real git (long-lived `rev-list` / `cat-file --batch` processes), output parsers, and `RepoSession` which owns per-repo state. No libgit2 — everything is plain git commands.
- `packages/graph-core` — pure layout logic: lane allocation and edge building for the commit graph. No I/O.
- `apps/server` — `node:http` + WebSocket (`ws`) server; hosts the oRPC router, serves the built client, agent-session detection (`agent-sessions.ts`, with `/proc` on Linux, `ps`/`lsof` on macOS, and native process metadata on Windows), GitHub CI status polling.
- `apps/client` — React 19 + Vite + Zustand. Key concept: **optimistic mutations** — `optimistic-graph.ts` predicts the post-mutation graph layout so checkout/merge/rebase/etc. animate immediately, then reconciles against the real server result; failures roll back.
- `apps/cli` — wraps the server into a standalone executable that embeds the bun runtime.

Client ⇄ server transport is a single WebSocket carrying oRPC calls. No database; all state is derived from the git repo itself.

Workspace packages resolve to `src/*.ts` directly under bun (see `exports` "bun" condition), so no build step is needed during development; `dist/` builds exist for the compiled CLI.

---
> Source: [capaj/ingit](https://github.com/capaj/ingit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
