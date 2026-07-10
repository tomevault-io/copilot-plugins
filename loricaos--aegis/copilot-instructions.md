## aegis

> provides and execs `/bin/vigil` as init (panicking "no init found" if there is

# CLAUDE.md — Aegis kernel

Guidance for working in this repo. Read it before making changes.

## What this repo is

Aegis is a **standalone, clean-slate x86-64 kernel** — just the kernel. It
embeds no userland: at boot it mounts the root filesystem the bootloader/OS
provides and execs `/bin/vigil` as init (panicking "no init found" if there is
none, like Linux). It publishes a versioned `aegis.elf` artifact that an OS
consumes; the reference OS is [LoricaOS](https://github.com/LoricaOS/LoricaOS),
which fetches `aegis.elf` rather than building it here.

## The security model is the product — never weaken it

Aegis enforces **no ambient authority**: no process, including `uid=0`, holds
power it was not explicitly granted. `uid=0` is cosmetic; authority comes only
from unforgeable capability tokens validated at the syscall boundary
(`cap_check` against the per-process cap table). Any change that lets a process
act without a capability it was granted — or that makes a capability ambient,
inheritable-by-default, or forgeable — is a regression in the one thing this
kernel exists to do. When in doubt, fail closed (return `ENOCAP`/`EFAULT`).

## Layout

```
kernel/arch/x86_64/   boot, SMP, IDT/ISR, context switch, LAPIC, paging
kernel/syscall/       the syscall surface (sys_*.c) — the trust boundary
kernel/mm/            PMM, VMM, uaccess (copy_to/from_user + exception table)
kernel/sched/         scheduler, processes, fork/exec
kernel/net/           netdev, eth, IP/TCP/UDP, sockets
kernel/core/          early init, cmdline parsing, panic
kernel/drivers/       NVMe, xHCI, PS/2, framebuffer, HDA, ...
test/                 in-tree checks
tools/                fetch-limine.sh (pinned bootloader for the smoke-test ISO)
```

## Build

x86_64-elf cross toolchain (gcc/ld/objcopy/nm) + `nasm` + `xorriso` and Limine
(`tools/fetch-limine.sh`) for the smoke-test ISO. **No Rust/cargo** — the
capability core is plain C (`kernel/cap/cap.c`); the kernel builds with a C
toolchain alone (a prerequisite for LoricaOS self-hosting).

```
make           # build/aegis.elf  (the shipped artifact)
make iso       # build/aegis.iso  — kernel-only smoke test, bootable via Limine
make test      # boot the kernel alone; the "no init found" panic is SUCCESS
make dist      # the stripped, release aegis.elf
```

`make iso`/`make test` are a **smoke test only** — there is no userland here, so
a clean boot ends at the "no init found" panic. Real boots happen in LoricaOS.

## Conventions

- Match the surrounding code: this is a pure C kernel (the capability core, once
  Rust, is now `kernel/cap/cap.c`). No new abstractions without a second caller;
  fail closed at the boundary.
- Kernel references that are legitimately "Aegis" (the name `aegis.elf`, the
  `/etc/aegis/*` capability paths, "the Aegis kernel") stay — the OS on top is
  "LoricaOS"; do not rebrand kernel identifiers.
- Commit trailer: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.

---
> Source: [LoricaOS/Aegis](https://github.com/LoricaOS/Aegis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
