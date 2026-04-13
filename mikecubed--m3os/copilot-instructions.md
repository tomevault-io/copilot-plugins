## m3os

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**m3OS** (technical name: `m3os`) is a toy bootable OS in Rust: microkernel architecture, x86_64, UEFI boot. Currently at kernel v0.51.0 with a functional userspace including init, login, shell, coreutils, networking, true SMP multitasking (per-core syscalls, per-CPU run queues, priority scheduling, load balancing, CPU affinity), persistent storage, signals, a text editor, multi-user accounts with permission enforcement, PTY (pseudo-terminal) pairs, a telnet server for remote shell access, an SSH server for encrypted remote access (using the sunset IO-less SSH library), build tools, a robust memory subsystem with buddy allocator, slab caches, demand paging, mprotect, and working munmap, real-time clock with wall-clock timekeeping, I/O multiplexing with wait-queue-driven poll/select/epoll and non-blocking I/O, Unix domain sockets for local IPC, kernel-level threading with clone(CLONE_THREAD), real futex wait/wake queues, thread groups, and per-thread TLS, a userspace cryptography library (SHA-256, HMAC, HKDF, ChaCha20-Poly1305, AES-256-CTR, Ed25519, X25519, CSPRNG) with sha256sum and genkey utilities, Rust cross-compilation support for musl-linked std programs (hello-rust, sysinfo-rust, httpd-rust, calc-rust, todo-rust) with a custom x86_64-m3os target spec, a BSD-style ports system for building and installing third-party software from source (Lua, zlib, bc, sbase, mandoc, minizip) with dependency resolution and package tracking, a data-driven service manager (enhanced init) with dependency-ordered boot, restart backoff with crash classification, per-service stop timeouts, directory-scanned service configs, system logging (syslogd via /dev/log Unix socket with init lifecycle integration), cron scheduling (crond with standard crontab format), orderly shutdown/reboot with orphan reaping via sys_reboot syscall, and administration commands (service list/status/start/stop/restart/enable/disable, logger, shutdown, reboot, hostname, who, w, last, crontab), capability-based IPC with grants, bulk-data transport, and ring-3-safe service registry.

## Build & Run

Uses the `xtask` pattern — always build through `cargo xtask`, never `cargo build` directly.

```bash
cargo xtask run          # build + launch in QEMU (headless, serial output)
cargo xtask run-gui      # build + launch in QEMU (GUI with framebuffer)
cargo xtask image        # build bootable disk image (UEFI raw + VHDX)
cargo xtask image --sign # build + sign EFI binary for Secure Boot
cargo xtask check        # clippy (-D warnings) + rustfmt + kernel-core host tests
cargo xtask fmt --fix    # auto-format all workspace source
cargo xtask test         # run all kernel tests in QEMU via ISA debug exit
cargo xtask test --test <name>  # run a single QEMU test binary
cargo xtask test --timeout 120  # custom timeout (default 60s)
cargo xtask test --display      # show QEMU window for debugging
cargo xtask sign         # sign EFI binary with Secure Boot keys
cargo test -p kernel-core       # run kernel-core host-side unit tests directly
```

Tests cannot use `cargo test` on the kernel — it is `no_std` and tests run inside QEMU via the xtask harness. Pure-logic code lives in `kernel-core` and is testable on the host via `cargo test -p kernel-core`.

## Git Workflow

All work must happen on a feature branch with a pull request to `main`. Never commit directly to `main`.

```bash
git checkout -b feat/my-feature       # 1. create feature branch
# ... make changes ...
git add <files> && git commit         # 2. commit
git push -u origin feat/my-feature    # 3. push
gh pr create --base main              # 4. open PR to main
# 5. user merges PR after review
```

Branch naming: `feat/`, `fix/`, `refactor/`, `docs/` prefixes as appropriate.

## First-Time Setup

After cloning, install the git hooks so quality gates run before commits and pushes:

```bash
./setup.sh
```

This sets `core.hooksPath` to `.githooks/`, which contains pre-commit and pre-push hooks that run `cargo xtask check`.

## Architecture

Microkernel: ring 0 kernel handles memory management, scheduling, IPC, interrupt routing, and device drivers. Userspace processes run in ring 3 and communicate through IPC and syscalls.

```
Ring 0 (kernel/):                Ring 3 (userspace/):
  - Frame allocator                - init (PID 1 daemon)
  - Page table manager             - sh0 (built-in shell)
  - Scheduler (SMP-aware)          - coreutils (cat, ls, grep, etc.)
  - IPC engine + capabilities      - ping (ICMP network tool)
  - IDT / APIC / interrupt router  - edit (text editor)
                                   - login, su, passwd, adduser
                                   - id, whoami
                                   - ion shell (external)
  - Syscall gate
  - VFS + FAT32 + tmpfs
  - Network stack (IPv4/TCP/UDP)
  - Unix domain sockets (AF_UNIX)
  - VirtIO drivers (blk, net)
  - ACPI / PCI enumeration
  - Framebuffer console
  - TTY + signal handling
  - SMP (multi-core boot + IPI)
```

### Workspace Crates

```
Cargo.toml                # workspace root (default-members = ["kernel"])
kernel/                   # main OS kernel binary (no_std)
kernel-core/              # shared library — host-testable pure logic (no_std + std feature)
xtask/                    # build system (host, std)
userspace/
  syscall-lib/            # syscall wrapper library for userspace Rust binaries
  exit0/                  # test binary: simple exit
  fork-test/              # test binary: fork behavior
  echo-args/              # test binary: argument echo
  init/                   # PID 1 init daemon
  shell/                  # sh0 shell (binary name: sh0)
  ping/                   # ICMP ping utility
  edit/                   # full-screen text editor (kibi-style)
  login/                  # login authentication (Phase 27)
  su/                     # switch user (Phase 27)
  passwd/                 # change password (Phase 27)
  adduser/                # create user account (Phase 27)
  pty-test/               # PTY subsystem test (Phase 29)
  unix-socket-test/       # Unix domain socket test (Phase 39)
  thread-test/            # Threading primitives test (Phase 40)
  crypto-lib/             # Cryptography library (Phase 42)
  crypto-test/            # Crypto integration test (Phase 42)
  telnetd/                # Telnet server daemon (Phase 30)
  sshd/                   # SSH server daemon (Phase 43)
  syslogd/                # System logging daemon (Phase 46)
  crond/                  # Cron scheduler daemon (Phase 46)
  coreutils/              # C implementations: cat, cp, echo, env, grep, id, ls, mkdir, mv, pwd, rm, rmdir, sleep, true, false, prompt, whoami, touch, stat, wc, ar, install
  coreutils-rs/           # Rust implementations: true, false, echo, pwd, sleep, rm, mkdir, rmdir, mv, touch, stat, wc, ar, install, meminfo, date, uptime, sha256sum, genkey, service, logger, shutdown, reboot, hostname, who, w, last, crontab
  demo-project/           # Multi-file C demo project for make testing (Phase 32)
  hello-c/                # C hello world test
  signal-test/            # C signal handling test
  stdin-test/             # C stdin test
  tmpfs-test/             # C tmpfs test
  # Phase 44: musl-linked Rust std programs (standalone crates, NOT workspace members)
  hello-rust/             # Rust std hello world (musl cross-compiled, Phase 44)
  sysinfo-rust/           # System info tool via std::fs (Phase 44)
  httpd-rust/             # Minimal HTTP server via std::net (Phase 44)
  calc-rust/              # Interactive calculator via std::io (Phase 44)
  todo-rust/              # Persistent todo list via std::fs (Phase 44)
```

### Ports Tree Layout (Phase 45)

```
ports/
  port.sh                 # port command (installed at /usr/bin/port)
  lang/lua/               # Lua 5.4.7 scripting language port
  lib/zlib/               # zlib 1.3.1 compression library port
  math/bc/                # bc calculator port
  core/sbase/             # suckless Unix tools port (basename, seq, rev, etc.)
  doc/mandoc/             # man page formatter port
  util/minizip/           # zlib-dependent test port
  <category>/<program>/
    Portfile              # metadata: NAME, VERSION, DESCRIPTION, CATEGORY, DEPS
    Makefile              # targets: fetch, patch, build, install, clean
    src/                  # bundled source code
    patches/              # m3OS-specific patches
```

### Kernel Source Layout

```
kernel/src/
  main.rs              # entry point, boot sequence
  serial.rs            # serial I/O + log backend
  pipe.rs              # inter-process pipes
  pty.rs               # PTY pair table and lifecycle (Phase 29)
  rtc.rs               # CMOS real-time clock driver (Phase 34)
  signal.rs            # POSIX-style signal handling
  stdin.rs             # stdin abstraction
  tty.rs               # TTY/terminal subsystem
  testing.rs           # QEMU test framework
  arch/x86_64/         # GDT, IDT (APIC-based), paging, syscall gate
  acpi/                # ACPI table parsing (RSDP, MADT)
  blk/                 # block devices: VirtIO-blk, MBR parsing
  fb/                  # framebuffer console driver
  fs/                  # VFS layer, FAT32, tmpfs, ramdisk, protocol
  ipc/                 # endpoints, capabilities, messages, notifications, registry
  mm/                  # buddy frame allocator, paging, heap, slab caches, user_space, ELF loader
  net/                 # IPv4, ARP, Ethernet, ICMP, TCP, UDP, Unix domain sockets, VirtIO-net, dispatch
  pci/                 # PCI device enumeration
  process/             # process management (fork, exec, exit, wait, threads, futex)
  smp/                 # AP boot, IPI, TLB shootdown
  task/                # scheduler (SMP-aware round-robin)
kernel/initrd/           # pre-built ELF binaries embedded as initial ramdisk
```

### kernel-core Source Layout

```
kernel-core/src/
  lib.rs               # module declarations
  types.rs             # shared types
  buddy.rs             # buddy frame allocator (Phase 33)
  slab.rs              # slab cache allocator (Phase 33)
  time.rs              # time conversion library (Phase 34)
  fb.rs                # framebuffer abstractions
  pipe.rs              # pipe abstractions
  pty.rs               # PTY pair state, ring buffers (Phase 29)
  tty.rs               # TTY abstractions
  fs/                  # FAT32, MBR, tmpfs abstractions
  ipc/                 # capability, message, registry abstractions
  net/                 # ARP, Ethernet, ICMP, IPv4, TCP, UDP abstractions
```

## Critical Conventions

### Target flags — do not remove

In `.cargo/config.toml` / target spec:
- `"disable-redzone": true` — hardware interrupts use the stack; removing this causes silent stack corruption
- `"-mmx,-sse"` — disables SIMD to avoid FPU state save/restore on context switches
- `"panic-strategy": "abort"` — no unwinding; panics halt the machine

### `no_std` everywhere in kernel and userspace

All crates under `kernel/` and `userspace/` are `#![no_std]`. Only use `alloc` types (`Vec`, `Box`, `Arc`) after heap initialization. `kernel-core` supports both `no_std` (kernel) and `std` (host tests) via feature flags.

### `unsafe` only at hardware boundaries

Acceptable only for: hardware register/port I/O, page table/GDT/IDT setup, `enter_userspace()`/`switch_context()` asm stubs, global allocator initialization, APIC/ACPI MMIO access, VirtIO ring manipulation. Always wrap in a safe abstraction immediately.

All crates use Rust **edition 2024** — the body of an `unsafe fn` is *not* implicitly unsafe. You must wrap unsafe operations in explicit `unsafe {}` blocks inside unsafe functions.

### IPC model — read the doc before touching `kernel/src/ipc/`

Synchronous rendezvous + async notification objects (seL4-style):
- Server-to-server: sync `call`/`reply_recv`
- IRQ/vsync: `Notification` objects (word-sized bitfield, safe to signal from interrupt handlers)
- Bulk data: page capability grants, never IPC payloads
- Userspace servers must never share writable memory

### Interrupt handlers

Do the minimum: read scancode / ack interrupt / push to ring buffer / send EOI. No allocation, no blocking, no IPC from within an interrupt handler.

### Capabilities

Integer index into the current process's `CapabilityTable`. Kernel validates every handle on every syscall. Transfer via `sys_cap_grant` — never forge or copy raw capability values.

### Syscall ABI

| Register | Role |
|---|---|
| `rax` | Syscall number (in) / return value (out) |
| `rdi`, `rsi`, `rdx`, `r10`, `r8`, `r9` | Arguments 1–6 |

`rcx` and `r11` are clobbered by `syscall` — never use them for arguments.

### Context switch

`switch_context(current, next)` saves/restores only callee-saved registers (`rbx`, `rbp`, `r12`–`r15`, `rsp`, `rip`). Do not change without auditing every call site.

### SMP conventions

- BSP (bootstrap processor) completes full kernel init before waking APs
- APs initialize their own GDT, IDT, APIC, and enter the scheduler idle loop
- Use IPI for TLB shootdown on page table updates affecting multiple cores
- Per-CPU data accessed via APIC ID — avoid global mutable state without proper locking

### QEMU test exit convention

```rust
// Write to I/O port 0xf4 (isa-debug-exit device)
// QEMU exit codes: 0x21 = success, 0x23 = failure
const QEMU_EXIT_SUCCESS: u32 = 0x10;
const QEMU_EXIT_FAILURE: u32 = 0x11;
```

### Userspace-first rule

New high-level policy defaults to userspace. Before adding policy-heavy code to ring 0, check the architecture review checklist in `docs/appendix/architecture-and-syscalls.md`.

### `BootInfo` is read-only after init

Parse memory regions, framebuffer, RSDP during `kernel_main` init and store in typed kernel structures. Do not hold long-lived references to `BootInfo`.

## Key Crates

| Crate | Purpose |
|---|---|
| `bootloader_api` | Kernel entry point macro, `BootInfo` |
| `x86_64` | `PageTable`, `IDT`, `GDT`, `PhysAddr`/`VirtAddr`, port I/O |
| `uart_16550` | Serial port driver — primary debug output |
| `pic8259` | 8259 PIC init and EOI |
| `linked_list_allocator` | `#[global_allocator]` for kernel heap |
| `spin` | `Mutex`/`RwLock` for `no_std` |
| `log` | Logging facade; backend writes to serial |
| `kernel-core` | Shared pure-logic library, host-testable |

## Documentation in `docs/`

Read before making significant changes:

| File | When |
|---|---|
| `docs/appendix/architecture-and-syscalls.md` | Orientation — kernel vs. userspace split, syscall ABI |
| `docs/02-memory.md` | Before touching frame allocator, page tables, or heap |
| `docs/06-ipc.md` | Before touching `kernel/src/ipc/` or syscalls |
| `docs/08-storage-and-vfs.md` | Before touching `kernel/src/fs/` or block devices |
| `docs/appendix/testing.md` | Before writing kernel tests or modifying the xtask harness |
| `docs/11-elf-loader-and-process-model.md` | Before touching ELF loading or process lifecycle |
| `docs/12-posix-compatibility-layer.md` | Before adding syscalls or POSIX behavior |
| `docs/16-network.md` | Before touching `kernel/src/net/` |
| `docs/19-signal-handlers.md` | Before touching signal delivery |
| `docs/22-tty-terminal.md` | Before touching TTY/terminal subsystem |
| `docs/25-smp.md` | Before touching SMP or multi-core code |
| `docs/26-text-editor.md` | Before touching the edit binary or userspace heap allocator |
| `docs/29-pty-subsystem.md` | Before touching PTY pairs, session management, or controlling terminals |
| `docs/30-telnet-server.md` | Before touching telnetd, socket refcounting, or network server architecture |
| `docs/32-build-tools.md` | Before touching make/pdpmake, ar, build utilities, or demo project |
| `docs/33-kernel-memory.md` | Before touching buddy allocator, slab caches, munmap, or meminfo |
| `docs/34-timekeeping.md` | Before touching RTC, clock_gettime, gettimeofday, or time conversion |
| `docs/roadmap/39-unix-domain-sockets.md` | Before touching Unix domain sockets, AF_UNIX, socketpair, or `kernel/src/net/unix.rs` |
| `docs/roadmap/42-crypto-primitives.md` | Before touching crypto-lib, sha256sum, genkey, or RustCrypto integration |
| `docs/roadmap/43-ssh-server.md` | Before touching sshd, sunset integration, host keys, or SSH authentication |
| `docs/43a-crash-diagnostics.md` | Before touching panic_diag, fault handler diagnostics, or scheduler/fork/IPC assertions |
| `docs/43b-kernel-trace-ring.md` | Before touching trace_ring, trace events, per-core trace rings, or sys_ktrace |
| `docs/43c-regression-stress-ci.md` | Before touching xtask regression/stress commands, CI workflows, or proptest/loom tests |
| `docs/roadmap/44-rust-cross-compilation.md` | Before touching musl Rust cross-compilation, xtask musl Rust builds, or custom target specs |
| `docs/roadmap/45-ports-system.md` | Before touching ports tree, port command, Portfile format, or xtask ports integration |
| `docs/roadmap/46-system-services.md` | Before touching init service manager, syslogd, crond, service command, or sys_reboot |
| `docs/appendix/sunset-local-fork.md` | Before modifying sunset-local/ or the sshd session event loop |
| `docs/roadmap/README.md` | Open design questions and per-phase scope |

Phase-specific roadmaps and task lists live in `docs/roadmap/` (phases 01–48) with corresponding `docs/roadmap/tasks/` breakdowns.

### Documentation templates — all docs must conform

All roadmap docs must follow the templates in `docs/appendix/doc-templates.md`. When creating or updating docs, use the matching template:

| Doc type | Template section | Required fields |
|---|---|---|
| Phase design doc | `docs/roadmap/NN-slug.md` | Status, Source Ref, Depends on, Builds on, Primary Components, Milestone Goal, Why This Phase Exists, Learning Goals, Feature Scope, Important Components and How They Work, How This Builds on Earlier Phases, Implementation Outline, Acceptance Criteria, Companion Task List, How Real OS Implementations Differ, Deferred Until Later |
| Phase task doc | `docs/roadmap/tasks/NN-slug-tasks.md` | Status, Source Ref, Depends on, Goal, Track Layout table, per-track sections with tasks containing File/Symbol/Why it matters/Acceptance, Documentation Notes |
| Roadmap README row | `docs/roadmap/README.md` | Phase, Theme, Primary Outcome, Status, Source Ref, Milestone link, Tasks link |

Rules:
- Never create a task doc without all template sections populated.
- Never create a design doc missing Status, Source Ref, Depends on, or Builds on.
- Task acceptance items must be concrete and measurable — no vague "works correctly".
- Each task must have File, Symbol, and Why it matters fields.
- Update the roadmap README row when creating or completing a phase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mikecubed)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mikecubed)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
