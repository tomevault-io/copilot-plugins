## opencode-plugin-controller

> opencode-plugin-controller is a monorepo for an **OpenCode TUI plugin** that makes the terminal UI navigable with a **game controller**.

opencode-plugin-controller is a monorepo for an **OpenCode TUI plugin** that makes the terminal UI navigable with a **game controller**.

Our priorities are (not ordered, all are important):

- Maintainability
- Reliability
- Performance

If a tradeoff is required, choose correctness and robustness over short-term convenience.

## Architecture (directional)

- **Gamepad router (Rust)** — `crates/gamepad-router`: exclusive grab of a source pad; forwards events to one active session-owned sink over a Unix control socket (JSON lines).
- **Gamepad sidecar (Rust)** — `crates/gamepad-sidecar`: sync `gilrs` loop, `next_event_blocking`, JSON lines on stdout, logs on stderr.
- **JS bridge** — `packages/gamepad` (`@ericc-ch/gamepad`): spawns sidecar, exposes raw gilrs events as an Effect `Stream`.
- **OpenCode plugin** — `packages/plugin` (`@ericc-ch/opencode-plugin-controller`): spawns `gamepad-router` on first controller connect, consumes the bridge, drives TUI navigation. Target the OpenCode **v2** plugin/SDK surface (`@opencode-ai/plugin` v2 entrypoints) even when the published package version is still `1.x`.

Use Bun as package manager (`bun install`, workspaces in root `package.json`).
Run first-party `.ts` with Bun (`bun path/to/file.ts`, `#!/usr/bin/env bun`). Use `node:` imports only, no Bun-specific runtime APIs except in compiled-binary build scripts.

Prefer e2e close to a real user: virtual gamepad → gamepad-router → plugin, with [terminal-control](https://github.com/anomalyco/terminal-control) as the screen checker when asserting OpenCode TUI behavior.

After completing a task, run:

- `bun run check` — typecheck per package, `vitest` at workspace root, lint at workspace root
- `cargo check` — when Rust under `crates/` changed (use `cargo clippy` if you want lints too)

For TypeScript style, follow the code-conventions skill.

See [PLAN.md](./PLAN.md) for full context and user preferences.

## Workspace

- `crates/gamepad-router` — exclusive pad forwarder (`cargo run -p gamepad-router`)
- `crates/virtual-gamepad` — virtual source pad for local e2e (`cargo run -p virtual-gamepad`)
- `crates/gamepad-sidecar` — gilrs sidecar binary (`cargo run -p gamepad-sidecar`)
- `packages/gamepad` — gilrs event stream bridge (`@ericc-ch/gamepad`)
- `packages/plugin` — OpenCode TUI plugin (`@ericc-ch/opencode-plugin-controller`, exports `./tui`)

## Subagents

When spawning Task subagents (review, explore, etc.), use Composer 2.5 / Composer 2.5 Fast depending on the task.

## References Directory

The `/tmp/references/` directory contains shallow clones of important external repositories (populated by `bun scripts/references.ts`).
Never make any changes in this directory — it is meant as reference only.

Prefer exploring and reading this directory over searching for documentation. Think of it as the source of truth.

Available references:

- effect — Effect (v4 on main)
- gilrs — GilRs gamepad library
- opencode — OpenCode
- opentui — OpenTUI

---
> Source: [ericc-ch/opencode-plugin-controller](https://github.com/ericc-ch/opencode-plugin-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
