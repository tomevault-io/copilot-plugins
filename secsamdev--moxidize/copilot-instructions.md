## moxidize

> Moxidize is a Rust workspace for desktop application resiliency testing and validation across platforms.

# Copilot Instructions for Moxidize

Moxidize is a Rust workspace for desktop application resiliency testing and validation across platforms.

## Repository Layout
- `agent/`: test agent runtime and platform actions (Windows/Linux), includes installer assets under `agent/wix/`.
- `common/`: shared domain and API models used by all binaries.
- `server/`: orchestration backend with actors/controllers/services architecture.
- `user/`: terminal user client for controlling tests and viewing logs.
- `xtask/`: build and packaging orchestration helpers.
- `scenarios/`: sample scenario definitions.

## Architecture Overview
- `user/` starts and controls scenario runs through the server, and can subscribe to agent or app logs while a run is active.
- `server/` is the orchestration center: it loads scenarios, expands them into executable tasks, assigns work to connected agents, and records run state.
- `agent/` maintains a persistent server connection, receives tasks over the shared agent API, executes platform-specific actions, and reports task results and logs back to the server.
- `common/` defines the contracts shared across binaries: scenario structure, action names, task/result types, authentication payloads, and the agent or user API messages.
- `xtask/` builds and packages the workspace binaries and installer assets; treat it as build plumbing rather than runtime behavior.

## Runtime Flow
- Scenario definitions are loaded from YAML into the shared `TestScenario` model in `common/`, then the server converts them into calculated runs and agent tasks.
- The server keeps active run state in the repository layer and uses the existing actor + controller + service split to coordinate user commands, agent connections, task scheduling, and log streaming.
- Agents poll for work, persist their current task locally, execute the mapped action implementation, and send `CompleteTask` messages back to the server when work finishes or fails.
- Most runtime behavior changes should preserve these boundaries: shared data contracts in `common/`, orchestration decisions in `server/services`, transport handling in server actors/controllers, and host-specific execution in `agent/actions` or `agent/services`.

## Product Naming
- Product name: `Moxidize`
- Agent binary/service: `moxidize-agent`
- Server binary: `moxidize-server`
- CLI binary: `moxidize-cli`
- Shared crate: `moxidize-core` (import path `moxidize_core`)

Prefer preserving this naming consistently in code, docs, logs, and installer assets.

## Build and Validation Commands
From repo root:
- `cargo check --workspace`
- `cargo build --workspace`
- `cargo xtask build-agent --target-dir "<path>"`
- `cargo xtask build-server --target-dir "<path>"`
- `cargo xtask build-user --target-dir "<path>"`
- `cargo xtask build-installer --target-dir "<path>"`

## Coding Expectations
- Keep changes scoped and avoid touching generated `target/` outputs.
- Update `Cargo.toml` and dependent imports together when crate names change.
- For Windows installer changes, keep WiX product/service/file names aligned with produced binaries.
- Preserve existing architecture patterns:
  - server: actor + controller + service separation
  - agent: action modules grouped by concern (service, machine, metrics, install)

## Documentation Expectations
When changing product identity or artifact names, update:
- root `README.md`
- relevant `Cargo.toml` package names/dependencies
- `agent/wix/main.wxs`
- any xtask build assumptions in `xtask/src/build/`

---
> Source: [SecSamDev/moxidize](https://github.com/SecSamDev/moxidize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
