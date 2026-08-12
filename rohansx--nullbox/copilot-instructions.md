## nullbox

> Generates atomic nftables rulesets. Blocks cloud metadata endpoints (AWS/GCP/Azure/DO/Alibaba), RFC-1918 ranges, and all link-local from agent interfaces. Per-agent allow rules are scoped to TAP devices. IPv6 blocked entirely from agents in v0.1. `blocklist.rs` has hardcoded CIDRs; `firewall.rs` generates the `nft -f` ruleset string. DNS resolver with IP binding is planned but not yet implemented.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

NullBox is an immutable, minimal Linux OS purpose-built for AI agents. It replaces systemd with `nulld` (PID 1 in Rust), isolates each agent in a libkrun microVM via `cage`, enforces default-deny networking via `egress`, and provides shared agent memory via `ctxgraph`. No SSH, no shell, no package manager, no mutable rootfs.

## Build Commands

```bash
# Build all workspace crates (host-native, for development)
cargo build --workspace

# Build for target (static musl binaries for the OS image)
cargo build --workspace --target x86_64-unknown-linux-musl --release

# Run all tests
cargo test --workspace

# Run tests for a single crate
cargo test -p nulld
cargo test -p cage
cargo test -p egress
cargo test -p ctxgraph

# Run a single test
cargo test -p ctxgraph -- store::tests::write_and_read

# Check without building
cargo check --workspace

# Clippy
cargo clippy --workspace
```

Release profile uses `opt-level = "z"`, LTO, strip, single codegen unit, and `panic = "abort"` — binaries must be as small as possible for the SquashFS rootfs.

## Workspace Architecture

Cargo workspace with 5 crates. Rust edition 2024, dual MIT/Apache-2.0 license.

### nulld (binary) — PID 1 init system
Replaces systemd entirely. Boot sequence: mount virtual filesystems → install signal handlers → load service config → start services in topological dependency order → main loop (reap zombies, restart failed services with exponential backoff, watch for shutdown signal). Wraps `main` in `catch_unwind` because PID 1 panic = kernel panic. Logs to `/dev/kmsg`. On fatal error, enters recovery hold (infinite sleep) instead of exiting.

Key modules:
- `mount.rs` — Ordered mount table (proc, sys, dev, devpts, shm, tmp, run, var). All mount flags are hardened (nosuid, nodev, noexec where appropriate).
- `service.rs` — `ServiceDef` type + Kahn's algorithm topological sort for dependency resolution. Restart policies: Always, OnFailure, Never.
- `supervisor.rs` — Process lifecycle management. Exponential backoff (1s→30s cap). Stops services in reverse dependency order with SIGTERM→5s→SIGKILL.
- `config.rs` — Reads `/system/config/nulld.toml`. Falls back to hardcoded builtin services (egress, ctxgraph, cage) when no config exists.
- `signal.rs` — SIGCHLD reaping via `waitpid(WNOHANG)`, SIGTERM/SIGINT → atomic shutdown flag.

### cage (library) — Per-agent microVM isolation
Each agent gets a dedicated microVM via libkrun. The `manifest.rs` module parses and validates `AGENT.toml` files which declare capabilities: allowed network domains, filesystem paths (must be absolute), credential refs, CPU/memory limits, tool risk levels (low/medium/high/critical). VM lifecycle modules (vm, network, filesystem, resources) are stubbed pending libkrun integration — this is the critical-path risk.

### egress (library) — Default-deny network controller
Generates atomic nftables rulesets. Blocks cloud metadata endpoints (AWS/GCP/Azure/DO/Alibaba), RFC-1918 ranges, and all link-local from agent interfaces. Per-agent allow rules are scoped to TAP devices. IPv6 blocked entirely from agents in v0.1. `blocklist.rs` has hardcoded CIDRs; `firewall.rs` generates the `nft -f` ruleset string. DNS resolver with IP binding is planned but not yet implemented.

### ctxgraph (library) — Shared agent memory graph
Content-addressed key-value store backed by SQLite. Hash = SHA-256(agent_id + key + canonical JSON value). Entries are immutable (INSERT OR IGNORE). Indexes on key, agent_id, and timestamp. Query interface: write → hash, read by hash, query by key prefix, history of a key, entries by agent. The `Graph` type in `query.rs` is the high-level API; will be exposed over Unix socket / virtio-vsock in the future.

### nullctl (binary) — CLI
Talks to nulld over Unix socket at `/run/nulld.sock`. JSON protocol. Commands: `agent start|stop|pause|resume <name>`, `status`. Future: swarm, snapshot, power, llm subcommands.

## Service Dependency Order

Default boot order (topological): egress → ctxgraph → cage (cage depends on both).
Configured via `/system/config/nulld.toml` with `[service.<name>]` sections specifying binary, args, depends_on, restart policy.

## Key Design Constraints

- **No dynamic linking.** All binaries must be statically linked (musl target). Zero `.so` dependencies.
- **No kernel modules.** CONFIG_MODULES=n. Everything compiled in.
- **Immutable rootfs.** SquashFS. Only `/var`, `/tmp`, `/run` are writable (tmpfs/overlay).
- **PID 1 must never exit.** A panicking nulld enters recovery hold. Exiting triggers `reboot()`.
- **Default-deny everything.** Network, filesystem, capabilities — all must be explicitly declared in AGENT.toml.
- **Content-addressed immutability.** ctxgraph entries cannot be modified once written.

## Development Environment

Requires: Rust nightly (edition 2024), Clang/LLVM, QEMU with KVM, musl toolchain (`rustup target add x86_64-unknown-linux-musl`), squashfs-tools, xorriso. Dev machine needs KVM and nested virtualization enabled for testing Cage microVMs inside QEMU.

## Current State

Phase 1 (v0.1) in progress. nulld, cage/manifest, egress, ctxgraph, and nullctl have working implementations with tests. Cage VM lifecycle (libkrun integration) is the critical-path blocker — currently stubbed. Kernel config, image build pipeline, and initramfs are not yet created. See `docs/PHASE1_PLAN.md` for the full implementation plan.

---
> Source: [rohansx/nullbox](https://github.com/rohansx/nullbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
