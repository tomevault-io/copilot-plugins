## savecraft-gg

> Savecraft parses video game save files and serves structured game state to AI assistants via MCP. Two components: a local Go daemon (WASM plugin runtime, filesystem watcher) and a remote Cloudflare Worker (MCP server, WebSocket push, SourceHub + UserHub Durable Objects).

# Savecraft

## Project Overview

Savecraft parses video game save files and serves structured game state to AI assistants via MCP. Two components: a local Go daemon (WASM plugin runtime, filesystem watcher) and a remote Cloudflare Worker (MCP server, WebSocket push, SourceHub + UserHub Durable Objects).

## Documentation

Read the doc relevant to your current task. Start with `overview.md` for orientation.

- `docs/overview.md` — What Savecraft is, system architecture, data flow, repo structure
- `docs/games.md` — Supported games: sections each connector produces, reference module descriptions
- `docs/daemon.md` — Go daemon: orchestrator, watcher, plugin loading, WebSocket client (`internal/`, `cmd/`)
- `docs/worker.md` — Cloudflare Worker: SourceHub + UserHub DOs, WebSocket protocol, D1 schemas (`worker/`)
- `docs/mcp.md` — OAuth architecture, MCP tools, notes, search, AI interaction patterns (`worker/src/mcp/`)
- `docs/views.md` — MCP Apps views: interactive UIs in host iframes, Svelte components, build pipeline (`views/`, `worker/src/mcp/views/`)
- `docs/view-design.md` — View design guide: when to build a view vs website, interaction patterns, visual principles
- `docs/plugins.md` — WASM plugin system, ndjson contract, signing, distribution (`plugins/`)
- `docs/plugin-development.md` — Local plugin dev workflow: build, auto-reload, staging, troubleshooting
- `docs/adapters.md` — API game adapters: server-side TypeScript modules for API-backed games (`worker/src/adapters/`)
- `docs/web.md` — SvelteKit frontend, onboarding, components (`web/`)
- `docs/infrastructure.md` — CI/CD, deployment, signing, installation, security (`.github/`, `install/`)
- `docs/roadmap.md` — Planned features: game adapters, reference data, monetization, game roadmap
- `docs/mcp-design.md` — Cross-platform MCP tool design best practices (external reference)

## Tech Stack

- **Daemon:** Go 1.26, wazero (WASM runtime), fsnotify, nhooyr.io/websocket
- **Cloud:** Cloudflare Workers (TypeScript), Durable Objects, R2, D1 (SQLite/FTS5)
- **Auth:** Clerk (OAuth, JWT, magic links)
- **Frontend:** SvelteKit, TypeScript
- **Plugins:** Go compiled to WASI Preview 1 (.wasm), ndjson stdout contract, Ed25519 signed
- **Protocol:** Binary protobuf on all WebSocket legs. `Message` (daemon↔server), `RelayedMessage` (server→browser). Schema: proto/protocol.proto → buf generate → Go + worker TS + web TS
- **Build:** just (Justfile), nix devenv + direnv

## Project Phase

**Pre-launch. Treat main as a feature branch.** No backwards compatibility required for anything — wire protocol, API contracts, DB schema, plugin format. Everything is subject to change. Delete old code, don't version it. This note will be removed when the project ships.

**Pre-launch does NOT mean cut corners.** Always prefer the architecturally correct solution. "No backwards compatibility" means you can freely change schemas, protocols, and APIs without migration shims — it does NOT mean quick hacks, skipping proper design, or lowering code quality. When presenting options, lead with the best architectural choice and explain why. You may also note a simpler alternative, but never default to it without asking.

## Key Conventions

- Monorepo, single Go module
- WebSocket protocol defined once in protobuf, codegen'd to Go + worker TS + web TS. Binary proto on the wire (`Message` for daemon↔server, `RelayedMessage` for server→browser). No mirrored types, no JSON on WebSocket.
- GameState types (plugin output) are hand-written Go/TS — section data is arbitrary JSON per game. Types live next to their consumers, not in grab-bag packages.
- Plugin stdout is ndjson: `{"type": "status"|"result"|"error", ...}` per line
- All daemon↔server communication over WebSocket + binary protobuf. Save data pushed as `PushSave` proto messages. No HTTP endpoints consumed by the daemon (except WASM plugin downloads).
- Save section data stored in D1 `sections` table, not R2. R2 used only for plugin WASM binaries.
- Save identity resolved by `(source_uuid, game_id, save_name)` → save UUID
- Sources own saves; users own sources. MCP/web access saves via source→user JOIN.
- Durable Objects use WebSocket Hibernation — no application-layer heartbeats
- Plugins provide a `summary` string for UI display (e.g. "Hammerdin, Level 89 Paladin")

## Worktrees

Use `just new-worktree <branch>` — do NOT run `git worktree add` directly. The recipe creates `.worktrees/<branch>` (gitignored), handles the new-vs-existing branch case, and mirrors the gitignored dev+build environment so checks/builds/tests work immediately with zero setup: every `node_modules` (worker, web, site, install/worker, views, reference) is symlinked from the primary checkout, built `*.wasm` is copied, and every `.env.local` is copied. `cd .worktrees/<branch> && direnv allow` after. Remove with `just rm-worktree <branch>` (force-removes the mirrored env; leaves the branch).

```
just new-worktree feature/my-branch
cd .worktrees/feature/my-branch && direnv allow
```

**Run parallel agents in separate worktrees, never the same checkout.** Two agents sharing one working tree corrupt each other's uncommitted state (files mutate mid-operation, diffs interleave). One `just new-worktree` per agent isolates working files; `.git` (refs, stashes, hooks) is still shared, so keep each agent on its own branch.

`node_modules` is symlinked (too large to copy) and therefore **shared** with the primary checkout: fine for editing, building, testing, and `just check`, but a `npm install`/dependency change mutates the shared tree — replace that subdir's `node_modules` symlink with a real `npm ci` first. `*.wasm` is copied (not symlinked), so rebuilding a plugin in a worktree is safe and cannot corrupt the primary checkout's artifacts.

`just new-worktree` symlinks `node_modules` (worker, web, site, install/worker, views, reference) rather than `npm ci`-ing each — instant, no multi-GB reinstall. The symlinks are shared with the primary checkout, which is fine for build/test/run; if you need to change dependencies in the worktree, replace that subdir's `node_modules` symlink with a real `npm ci`. Remove a worktree with `just rm-worktree feature/my-branch`.

## Development Principles

- TDD: write the test first, watch it fail, implement, watch it pass.
- No mocking libraries — hand-written fakes that implement the same interface.
- Tests assert behavior, not implementation details.
- Domain-specific conventions (Go, TypeScript, D2R, etc.) are in `.claude/skills/`. They load automatically when you're working in the relevant area.

## Important Paths

- `docs/` — Architecture docs (see Documentation section above)
- `proto/savecraft/v1/protocol.proto` — Canonical WebSocket protocol definition
- `internal/proto/savecraft/v1/` — Generated Go protobuf code (do not edit)
- `worker/src/proto/savecraft/v1/` — Generated TypeScript protobuf code (do not edit)

## Scripts

- `web/scripts/screenshot.ts` — Capture Storybook story screenshots via Playwright. Usage: `cd web && npx tsx scripts/screenshot.ts --grep <pattern>` or `--all`. Requires Storybook running. Output in `web/screenshots/`.
- `scripts/create-test-vm.sh` — Create a Windows 11 Pro Azure VM for manual testing (MSI install, WebView2 dialog, tray). Configures RDP for non-Windows clients. Delete with `az group delete --name savecraft-test-rg --yes --no-wait`.

## Commands

All targets in `Justfile`. Run `just --list` for full list.

- `just proto` — Generate Go + TypeScript from protobuf
- `just proto-lint` — Lint protobuf definitions
- `just test` — Run all tests (Go + Worker)
- `just test-go` — Run Go tests
- `just test-go-race` — Run Go tests with race detector
- `just test-worker` — Run Worker tests (Vitest + Miniflare)
- `just dev-worker` — Start Worker dev server (Miniflare)
- `just lint-go` — Run staticcheck + go vet
- `just fmt-go` — Run goimports
- `just check` — Lint, generate, test everything

---
> Source: [joshsymonds/savecraft.gg](https://github.com/joshsymonds/savecraft.gg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
