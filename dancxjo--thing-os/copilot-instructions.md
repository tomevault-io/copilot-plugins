## thing-os

> This file is a quick map of the repository so agents (and humans) can orient fast.

# Thing-OS Agent Notes

This file is a quick map of the repository so agents (and humans) can orient fast.

## What this repo is
- Thing-OS is a Rust workspace that builds a graph-based OS kernel plus userland apps.
- Build/test automation lives in `xtask` and is surfaced via `just`.

## Common commands
- Build ISO: `just iso` (override arch with `KARCH=aarch64`, `riscv64`, `loongarch64`)
- Run QEMU: `just run`
- Run BDD tests: `just behave` (see `tools/bdd`)
- Clean: `just clean`
- Audit platform boundary: `python3 scripts/audit_platform_boundary.py`

## Top-level layout (what's what)
- `abi/`: shared ABI types and syscalls between kernel/userspace.
- `bran/`: core kernel runtime (boot/runtime abstraction).
- `kernel/`: kernel crate and core kernel logic.
- `drivers/`: hardware driver crates.
- `userspace/`: user programs and demos (each subdir is a crate).
- `bloom/`, `blossom/`, `display/`: graphics/compositor-related crates.
- `stem/`, `stem-macros/`: internal libs and proc-macros.
- `targets/`: custom JSON target specs for bare metal builds.
- `tools/`: auxiliary tooling (BDD, pciids, etc).
- `xtask/`: build orchestration used by `just`.
- `docs/`: documentation and test reports (`docs/behavior/` is generated).
- `vendor/`: vendored dependencies (Limine, OVMF).

## Where to start when changing behavior
- Kernel interfaces: `abi/` and `kernel/`
- Syscall surface: `abi/src/syscall.rs`
- User apps: `userspace/`
- Build/config: `justfile`, `xtask/`, `targets/`

## Platform Layer Contract ("stem is our std")

**Thing-OS does not use Rust's `std`**. Instead:

- Kernel and userspace use `core` + `alloc` + `stem`
- Platform capabilities are explicit in `stem::pal` (Platform Abstraction Layer)
- Build tools (`xtask`, `tools/*`) can use `std` (they run at compile-time only)

**Key rules:**
- All kernel/userspace crates MUST have `#![no_std]`
- Platform primitives go in `stem::pal` (log, clock, abort, alloc)
- Run `python3 scripts/audit_platform_boundary.py` to verify compliance

**See `docs/platform.md` for the complete platform layer contract.**

## Notes
- Workspace members are listed in `Cargo.toml`.
- `target/` is build output and can be ignored in reviews.
- Reminder: use the `apply_patch` tool directly for file edits (avoid running it via exec). 
- Reminder: hashing helpers must use the correct byte width for each integer type (u32/i32 = 4 bytes).
- The `stem` build script expects `assets/pci/pci.ids`; ensure it exists (or skip builds that trigger `stem`'s build.rs) when running `cargo test`.
- Reminder: avoid clearing all UI caches on watch events; prefer targeted invalidation once event payloads provide node or bytespace IDs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dancxjo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dancxjo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
