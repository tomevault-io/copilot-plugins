## autosk

> This file provides guidance when working with code in this repository.


# autosk - task manager and workflow manager for coding agents

This file provides guidance when working with code in this repository.

## Architecture in one paragraph

autosk is split into a **Go front end** (the `autosk` CLI + the `autosk lazy`
TUI) and a **Bun/TypeScript daemon** (`autoskd`). The Go binary is a pure
JSON-RPC client — it owns no storage and links no database; it auto-spawns and
talks to `autoskd` over a Unix socket (proto-v2). `autoskd` (under `daemon/`)
is the sole owner of a project's `.autosk/` directory: tasks, comments, and
session transcripts live as files (there is **no database**), and workflows +
agents are **code** registered by extensions. The Tauri desktop GUI (`gui/`)
is a third front end — also a pure JSON-RPC client of `autoskd`.

There is no Rust workspace and no embedded database engine anywhere in this repo
any more (the only `cargo` consumer is the standalone Tauri backend,
`gui/src-tauri`).

## Build, test, lint

The Go binary builds CGO-free with plain `go build` / `go test` — no build tags,
no native libraries.

- `make build` — compiles `bin/autosk` (entrypoint: `cmd/autosk`), CGO-free
- `make build-autoskd` — compiles the Bun daemon into `bin/autoskd` with
  `bun build --compile`; the compiled binary embeds the Bun runtime, so **no
  global `bun` is needed at runtime**. There are no bundled extensions: the
  reference `@autosk/feature-dev` workflow is an npm package the daemon installs
  on first run (`ensureGlobalBootstrap`)
- `make test` — builds `autoskd` first, then runs `go test ./...` with
  `AUTOSKD_BIN` pointed at the compiled daemon (the verb-test harness seeds its
  own temp `~/.autosk/extensions/` fixture + an empty `settings.json`, so no
  real `npm install` runs)
- `make test-short` — same with `-short`
- `make vet` — `go vet ./...`
- `make fmt` — `gofmt`
- `make lint` — `golangci-lint run ./...` (requires `golangci-lint` installed)

The `cmd/autosk` verb tests are RPC clients: they need a live `autoskd` on disk,
which they **auto-spawn** after locating it via `$AUTOSKD_BIN`. A direct
`go test ./...` works without any tag, but the verb tests `t.Skip` unless
`autoskd` has been built (`make build-autoskd`) and `$AUTOSKD_BIN` is set —
which is why `make test` builds it first.

### Bun daemon workspace (`daemon/`)

`autoskd` and its public SDK + shipped extensions live in a Bun workspace under
`daemon/`. Checks run from `daemon/`:

- `bun install` — install + symlink the workspace packages (`bun install
  --frozen-lockfile` in CI)
- `bun run typecheck` — `tsc --noEmit` across every workspace package
- `bun test` — every package's `*.test.ts` (pure unit tests; no spawned daemon)
- `bun build --compile core/src/index.ts --outfile <path>` — produce the
  standalone daemon binary (`make build-autoskd` / `scripts/package-autoskd.sh`
  wrap this)

### Tauri GUI (`gui/`)

The desktop app is a React/Vite front end over a thin Tauri (Rust) backend that
is a **pure JSON-RPC client of `autoskd`** — `gui/src-tauri` is a standalone
cargo crate (there is no root cargo workspace) and links no storage engine.
Front-end checks run from `gui/`:

- `npm ci` — install (uses `gui/package-lock.json`)
- `npm run typecheck` — `tsc --noEmit` **plus** the
  `scripts/check-ipc-discipline.mjs` guard (one `invoke` site in
  `src/services/ipc.ts`, one `listen` site in `src/services/events.ts`)
- `npm run build` — `tsc && vite build` (production bundle)
- `npm run test` — `vitest` (pure reducer/selector/ipc logic; no browser or daemon)

Backend check runs from `gui/src-tauri/`:

- `cargo check` — typecheck the Tauri backend (a self-contained crate)

### Marketing site (`website/`)

The landing page is a static **Astro + Tailwind v4** site (dark-first, reuses the
GUI brand tokens) with **no backend** — `npm run build` emits a plain `dist/`
deployable to any static host. Checks run from `website/`:

- `npm install` — install (uses `website/package-lock.json`)
- `npm run dev` — `astro dev` at http://localhost:4321
- `npm run build` — `astro build` → `website/dist/`
- `npm run check` — `astro check` (type/diagnostics)
- `npm run preview` — serve the built `dist/` locally

App icons: sources live in `gui/src-tauri/icons/src/` (`autosk.icon` +
`autosk.png`) and all binary icon artifacts under `icons/` are stored in Git LFS
(run `git lfs install` after cloning). Regenerate the Liquid Glass `Assets.car` +
flat `.icns`/`.ico`/PNG fallback with `scripts/update-gui-icons.sh` (no args =
repo sources; pass `<App.icon> <icon-1024.png>` to adopt a new icon); full usage
is in the script header.

## Repo layout

- `cmd/autosk/` — Cobra CLI entrypoint (proto-v2 RPC client)
- `internal/lazy/` — lazygit-style TUI (`autosk lazy`), built on `jesseduffield/gocui`
- `internal/daemon/` — JSON-RPC client of `autoskd` (`rpcclient`, auto-spawn)
  plus the shared wire/view types (`api`, `runstore`) mirrored from
  `daemon/sdk/src/proto.ts`
- `internal/projectdb/`, `internal/store/` — `.autosk/` directory resolution
  (walk-up; no DB file) + the task/workflow/agent/session view types the front
  ends render over RPC
- `daemon/` — the Bun/TypeScript daemon workspace:
  - `daemon/sdk/` — `@autosk/sdk`: the public, extension-facing types
    (Task / Session / Workflow / Agent / `AutoskAPI`), the pi-format
    transcript entry types, and the **proto-v2 wire types** — the single source
    of truth the Go and Tauri clients mirror
  - `daemon/core/` — `@autosk/core`: the daemon binary (file store, project
    manager, extension loader, engine/scheduler, JSON-RPC server)
  - `daemon/extensions/sandbox/` — `@autosk/sandbox`: the agent-owned sandbox
    library (structural `Sandbox` + `worktreeSandbox()` / `dockerSandbox()` /
    `sandboxCleanupStep()`; absorbs the retired `@autosk/worktree` + `@autosk/docker`)
  - `daemon/extensions/pi-agent/` — `@autosk/pi-agent`: the shipped agent that
    drives `pi --mode rpc`
  - `daemon/extensions/claude-agent/` — `@autosk/claude-agent`: the shipped agent
    that drives Claude Code (`claude -p` headless stream-json); its tool surface
    is the per-session host HTTP MCP server the engine mints (`ctx.newMCPServer()`,
    advertised via `--mcp-config type:"http"`; the standalone `autoskd mcp` stdio
    server survives for external use)
  - `daemon/extensions/feature-dev/` — `@autosk/feature-dev`: the reference
    workflow (`dev → review → docs → validator → accept`), published to npm and
    installed by the daemon's first-run bootstrap
- `gui/` — the Tauri desktop app: `src/` (React + Vite front end), `src-tauri/`
  (thin Tauri backend, a pure JSON-RPC client of `autoskd`; standalone cargo crate)
- `website/` — the marketing landing page: a static Astro + Tailwind v4 site
  (dark-first, reuses the GUI brand tokens), no backend; `npm run build` →
  `website/dist/`, deployable to any static host (see `website/README.md`)
- `scripts/` — `package-autoskd.sh` (release packaging of the `autoskd` binary),
  `publish-extensions.sh` (publish the `@autosk/*` extension packages to npm),
  `clean-layout-test.sh` (the clean-machine auto-spawn smoke),
  `changelog-section.sh`, `pi-rpc-probe.sh`, `update-gui-icons.sh`

## Additional docs

- `docs/plans/yyyymmdd-*.md` — design plans for feature development
- `docs/concepts.md` — core concepts & data model (task model, status machine + ready set, blockers, metadata/`step_visits`, comments, `.autosk/` layout + hybrid ownership)
- `docs/cli.md` — the `autosk` CLI reference (global behavior, every verb + flag, env vars, scripting recipes)
- `docs/daemon.md` — `autoskd`, the Bun daemon (JSON-RPC, auto-spawn, sole owner of `.autosk/`)
- `docs/daemon-tutorial.md` — learn-by-doing: run your own daemon, watch a live session, serve many projects, expose it over TCP
- `docs/workflows.md` — code-based workflows + agents (the `@autosk/sdk` contracts)
- `docs/shipped.md` — catalog of the shipped workflows (`feature-dev`(`-cc`/`-docker`), `merge-to-current`) and agents (`pi-agent`, `claude-agent`): reference + how-to + tutorial
- `docs/extensions.md` — the extension system (discovery, factory, AutoskAPI, bundled extensions)
- `docs/extensions-tutorial.md` — learn-by-doing: write, run, break & recover your first extension
- `docs/extensions-tutorial-claude.md` — learn-by-doing: a practical Claude Code workflow (claude-agent + worktree)
- `docs/lazy.md` — the `autosk lazy` interactive TUI
- `gui/README.md` — the Tauri desktop GUI (architecture, scripts, local vs remote modes)
- `daemon/README.md` — the Bun daemon workspace overview
- `website/README.md` — the marketing landing page (Astro + Tailwind stack, dev/build/preview commands, structure)

## Conventions

- All text in tasks / comments / docs should be in English.
- Commits follow Conventional Commits: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`, etc.
- **Changelog policy.** Every PR that ships a user-visible change (CLI verb, flag, TUI hotkey, output format, env knob, …) updates `CHANGELOG.md` in the same PR. Format follows [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/): add or extend the top-of-file `## [Unreleased]` block with `### Added` / `### Changed` / `### Fixed` / `### Removed` / `### Deprecated` / `### Security` subsections; release tagging promotes the block to a dated `## [X.Y.Z] - YYYY-MM-DD` header. Keep each entry concise: **15 words maximum** per bullet. `CHANGELOG.md` at repo root is a symlink to `internal/changelog/CHANGELOG.md` (the canonical file the binary `go:embed`s for the lazy first-run popup) — edit either path, they resolve to the same file. Internal-only churn (refactors, test-only changes, build-system tweaks with no operator-visible effect) does NOT need an entry.
- Go 1.25 toolchain (`go.mod`); the Bun daemon targets the Bun runtime (TypeScript, ESM, `node:*` builtins).
- The TUI uses the `jesseduffield/gocui` fork (the lazygit one), not `awesome-gocui`.
- **Time rendering.** Anything user-facing on the Go side (CLI text output, TUI panes, flash toasts, command log) MUST format through `internal/timeformat` — `FormatDate` / `FormatTime` / `FormatDateTime` / `FormatDateTimeSmart`. These render in the operator's local timezone using the layouts `2006-01-02`, `15:04:05`, `2006-01-02 15:04:05`. Machine wire formats stay on **RFC3339 UTC** and must NOT route through `timeformat`: the JSON CLI output behind `--json`, the proto-v2 wire types defined in `daemon/sdk/src/proto.ts` and mirrored by the Go view types in `internal/daemon/api`, the on-disk file formats and transcript/comment-prompt rendering in `daemon/core`, and the SDK/transcript types in `daemon/sdk/src`.

---
> Source: [wierdbytes/autosk](https://github.com/wierdbytes/autosk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
