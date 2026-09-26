## hivebox

> HiveBox is a native Linux sandboxing system. Each **hivebox** is a lightweight isolated environment using kernel primitives (namespaces, cgroups, seccomp, Landlock). Written in Rust, single static binary.

# HiveBox — Agent Guide

## Project Overview

HiveBox is a native Linux sandboxing system. Each **hivebox** is a lightweight isolated environment using kernel primitives (namespaces, cgroups, seccomp, Landlock). Written in Rust, single static binary.

## Architecture

```
src/
├── main.rs                 # Entry point, CLI dispatch
├── cli/mod.rs              # CLI definition (clap)
├── api/
│   ├── mod.rs              # Axum router, auth middleware, server startup
│   ├── handlers.rs         # REST endpoint handlers
│   ├── types.rs            # Request/response types
│   ├── mcp.rs              # MCP over HTTP endpoint (JSON-RPC 2.0, 15 tools)
│   └── dashboard.rs        # Embedded web dashboard (single HTML page in raw string)
├── sandbox/
│   ├── mod.rs              # Orchestrator, SandboxConfig, SandboxState, create_and_run()
│   ├── manager.rs          # SandboxManager — persistent hivebox lifecycle, exec, cwd tracking
│   ├── filesystem.rs       # Rootfs prep (squashfs + overlayfs), pivot_root, mount specials
│   ├── namespace.rs        # Linux namespace creation (clone/unshare)
│   ├── cgroup.rs           # cgroup v2 resource limits (memory, CPU, PIDs)
│   ├── network.rs          # veth pairs, bridges, NAT, IP allocation
│   ├── seccomp.rs          # seccomp-BPF syscall filtering
│   ├── landlock.rs         # Landlock filesystem restrictions
│   └── capabilities.rs     # Capability dropping, NO_NEW_PRIVS
├── runtime/
│   ├── mod.rs              # ExecResult struct
│   ├── exec.rs             # Command execution, pipe management
│   └── cleanup.rs          # Resource cleanup
└── images/
    ├── mod.rs              # Image store management
    └── builder.rs          # Image building
```

## Key Concepts

- **Hivebox** (not "sandbox" in user-facing text): an isolated Linux environment
- **API routes** use `/api/v1/hiveboxes/...`
- **Dashboard**: embedded in `dashboard.rs` as a raw HTML string — no separate frontend build
- **Filesystem isolation**: overlayfs (squashfs lower + tmpfs upper), falls back to `cp -a` in Docker
- **Exec**: uses `nsenter` + `chroot` into the hivebox rootfs, with server-side cwd tracking
- **Primary color**: `#eac01b` (golden yellow)

## Build & Run

```bash
# Linux only — uses kernel primitives
cargo build --release --target x86_64-unknown-linux-musl

# Docker (for development on non-Linux)
docker compose up -d --build    # builds and starts on port 7070
docker compose logs -f          # follow logs
```

## API Endpoints

All under `/api/v1/hiveboxes`. Auth via `Authorization: Bearer <key>` header.

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/hiveboxes` | Create hivebox |
| GET | `/api/v1/hiveboxes` | List hiveboxes |
| GET | `/api/v1/hiveboxes/:id` | Get hivebox details |
| POST | `/api/v1/hiveboxes/:id/exec` | Execute command |
| POST | `/api/v1/hiveboxes/:id/mcp` | MCP over HTTP |
| PUT | `/api/v1/hiveboxes/:id/files` | Upload file |
| GET | `/api/v1/hiveboxes/:id/files` | Download file |
| DELETE | `/api/v1/hiveboxes/:id` | Destroy hivebox |
| GET | `/api/v1/analytics` | Analytics data |

## Conventions

- Use "hivebox" (not "sandbox") in all user-facing text, API paths, and documentation
- Internal Rust code still uses `Sandbox*` struct names (e.g., `SandboxManager`, `SandboxConfig`)
- Dashboard is a single embedded HTML page — edit `src/api/dashboard.rs`
- Logo SVG: `assets/logo.svg`
- The project compiles only on Linux (uses nix, libc, seccomp crates)
- Docker container runs privileged with `--cgroupns=host`

---
> Source: [TetiAI/hivebox](https://github.com/TetiAI/hivebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
