## atlas-runtime

> **Core rule: Keep Atlas-runtime small. Resist turning it into a framework.** Implement only current requirements. Do not solve hypothetical future problems.

# Atlas Runtime - Agent Instructions

**Core rule: Keep Atlas-runtime small. Resist turning it into a framework.** Implement only current requirements. Do not solve hypothetical future problems.

## Architecture

Use Go, stdlib `net/http`, Firecracker microVMs, and systemd. Run one process per VM. State is `config.toml` + `metadata.json` only. No database, no controller, no reconciler.

```text
config.toml
    ↓
systemd
    ↓
atlas-runtime
    ↓
Firecracker
```

Do not add Redis, PostgreSQL, ORM layers, message brokers, Raft, gossip, consensus, replication, leader election, distributed locks, event sourcing, or worker systems unless a real requirement demands them.

## Code quality

- Follow **ASD-STE100 Simplified Technical English** in comments and documentation.
- Keep names clear; avoid unnecessary abbreviations.
- Keep functions **under 50 lines**; prefer 25 or fewer when practical.
- Keep cyclomatic complexity **below 8**.
- Prefer simple control flow and early returns.
- Avoid deep inheritance, unnecessary interfaces, design patterns, and abstraction layers.
- A small amount of duplication is better than unnecessary abstraction.
- Prefer the standard library and existing dependencies.
- Add comments only when the reason or behavior is not obvious to a reader.
- Never add comments at the top of a file.

## Filesystem layout

Keep it flat. No `pkg/`, `internal/`, `domain/`, `service/`, `repository/`, `handler/`, `controller/`, `manager/`, `factory/`.

```text
atlas-runtime/
├── main.go
├── config.go
├── metadata.go
├── runtime.go
├── firecracker.go
├── network.go
├── console.go
├── api.go
├── image.go
├── systemd/
│   ├── atlas-vm@.service
│   └── atlas-generator
├── scripts/
│   └── build-image
├── Makefile
└── docs/
```

## State

Persistent state lives in the machine directory only:

- `config.toml` - user configuration
- `metadata.json` - persistent Atlas machine state + guest metadata
- `vpc-members.json` - cached remote VMs of this VPC plus the last beacon revision, to rebuild routes and resume the watch after a reboot with beacon down
- `rootfs` - machine disk
- `console.log` - console history
- `runtime/` - disposable runtime state (can be destroyed at any time)

## Important invariants

- `[boot]` is creation-time configuration. After `initialized=true`, `image`/`snapshot` are not consumed on normal start.
- `desired_state` is persisted atomically **before** acting on it.
- `config.toml` is the persistent authority. The runtime's in-memory config is not.
- If Linux, systemd, or Firecracker already solves it, Atlas-runtime should not invent another implementation.

## Development

Before adding code, ask: Is it required now? Is there a simpler solution? Can existing data structures solve it? Does it add a dependency, process, abstraction, or subsystem?

If it is only "useful later", do not add it. Keep changes small and focused. Do not refactor working code without a current reason. Test real behavior.

**When choosing between a simple design and a sophisticated design, choose the simple design.**

---
> Source: [tanmoysrt/atlas-runtime](https://github.com/tanmoysrt/atlas-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
