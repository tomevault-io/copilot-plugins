## fajaros-x86

> > Auto-loaded by Claude Code. Single source of truth for this repo.

# CLAUDE.md — FajarOS Nova v1.0.0 "Genesis" (x86_64)

> Auto-loaded by Claude Code. Single source of truth for this repo.

## Project Identity

- **Project:** FajarOS Nova v1.0.0 — x86_64 operating system written 100% in Fajar Lang
- **Codename:** "Nova" / "Genesis"
- **Target:** Intel Core i9-14900HX (Lenovo Legion Pro)
- **Language:** 100% Fajar Lang (`.fj` files)
- **Compiler:** [Fajar Lang](https://github.com/fajarkraton/fajar-lang) (`fj` binary)
- **Author:** Fajar (TaxPrime / PrimeCore.id)
- **Model:** Claude Opus 4.6 exclusively
- **LOC:** 11,615 lines across 35 modular `.fj` files
- **Commands:** 160+ shell commands

## Build System

Multi-file concatenation: Makefile cats 35 `.fj` files in dependency order into `build/combined.fj`, then compiles with `fj build --target x86_64-none`.

```bash
make build          # Concatenate + compile kernel ELF
make run            # QEMU serial (no KVM)
make run-kvm        # QEMU with KVM
make run-vga        # QEMU with VGA display
make run-smp        # QEMU with 4 cores
make run-nvme       # QEMU with NVMe disk
make run-net        # QEMU with networking
make debug          # QEMU + GDB (port 1234)
make iso            # Bootable GRUB2 ISO
make test           # Auto-exit test run
make loc            # Lines of code per file
make clean          # Remove build artifacts
```

## Architecture

```
Layer 5: Applications (@safe)    — Ring 3 — user programs, login, neofetch
Layer 4: Shell (@kernel)         — 160+ commands, scripting, history
Layer 3: Filesystem (@kernel)    — VFS, FAT32, RamFS, /dev, /proc
Layer 2: Drivers (@kernel)       — Serial, VGA, KB, PCI, NVMe, Net, USB, GPU
Layer 1: Microkernel (@kernel)   — MM, scheduler, IPC, syscall, SMP, ELF
Layer 0: Hardware                — Intel i9-14900HX, RTX 4090, NVMe
```

## Context Annotations

```
@kernel  → Ring 0, asm!/port I/O/MMIO, no heap strings, no tensor
@device  → Compute, tensor/AVX2/GPU, no raw pointers, no IRQ
@safe    → Ring 3, syscalls/strings, no direct hardware
```

## Module Map (Concatenation Order)

| # | File | LOC | Purpose |
|---|------|-----|---------|
| 1 | kernel/boot/constants.fj | 71 | VGA/color constants, security status |
| 2 | kernel/mm/frames.fj | 220 | Bitmap frame allocator (32768 frames) |
| 3 | kernel/mm/paging.fj | 118 | 4-level page tables, map/unmap |
| 4 | kernel/mm/heap.fj | 164 | Freelist heap (kmalloc/kfree) |
| 5 | kernel/mm/slab.fj | 103 | Slab allocator (power-of-2) |
| 6 | kernel/ipc/message.fj | 112 | IPC message queue |
| 7 | kernel/ipc/pipe.fj | 166 | Pipes + FD table |
| 8 | kernel/sched/process.fj | 127 | Process table v2 (16 PIDs) |
| 9 | kernel/sched/scheduler.fj | 67 | Init/shutdown |
| 10 | kernel/sched/smp.fj | 296 | SMP (INIT-SIPI-SIPI, AP trampoline) |
| 11 | kernel/sched/spinlock.fj | 34 | Test-and-set spinlock |
| 12 | kernel/interrupts/lapic.fj | 25 | LAPIC/IOAPIC constants |
| 13 | kernel/interrupts/timer.fj | 21 | sleep_ms, delay_us |
| 14 | kernel/syscall/entry.fj | 201 | SYSCALL stub + MSRs |
| 15 | kernel/syscall/dispatch.fj | 243 | Syscall table (8 syscalls) |
| 16 | kernel/syscall/elf.fj | 116 | ELF64 parser + loader |
| 17 | kernel/security/limits.fj | 218 | Resource limits, LAPIC commands |
| 18 | kernel/security/hardening.fj | 57 | FPU, Ring 3 status, RDRAND |
| 19 | drivers/serial.fj | 81 | 16550 UART (COM1) |
| 20 | drivers/vga.fj | 106 | VGA console engine |
| 21 | drivers/keyboard.fj | 202 | PS/2 keyboard + shift/caps |
| 22 | drivers/pci.fj | 181 | PCI enumeration + lspci |
| 23 | drivers/nvme.fj | 645 | NVMe block device |
| 24 | drivers/virtio_blk.fj | 44 | VirtIO block |
| 25 | drivers/virtio_net.fj | 393 | Network (Eth/ARP/IPv4/ICMP) |
| 26 | drivers/xhci.fj | 196 | USB XHCI detection |
| 27 | drivers/gpu.fj | 60 | GPU detection |
| 28 | fs/ramfs.fj | 115 | RAM filesystem |
| 29 | fs/fat32.fj | 752 | FAT32 filesystem |
| 30 | fs/vfs.fj | 325 | VFS (/dev, /proc, mount table) |
| 31 | shell/commands.fj | 3394 | 160+ commands + dispatch |
| 32 | shell/scripting.fj | 89 | Script execution |
| 33 | apps/user_programs.fj | 262 | Ring 3 programs + login |
| 34 | apps/mnist.fj | 6 | MNIST (reserved) |
| 35 | kernel/main.fj | 259 | kernel_main() entry + shell loop |

## Memory Map

```
0x000000 - 0x0FFFFF:  Reserved (1 MB)
0x100000 - 0x300000:  Kernel .text/.rodata/.data/.bss (2 MB)
0x400000 - 0x5FFFFF:  Frame bitmap (2 MB)
0x600000 - 0x6FFFFF:  Process table + command buffer + state (1 MB)
0x700000 - 0x7FFFFF:  Stack (1 MB, guard page at bottom)
0x800000 - 0x81FFFF:  SYSCALL entry stub (at 0x8200)
0x2000000:            User program (Ring 3)
0x2F00000:            User stack (Ring 3)
0x7000000 - 0x7FFFFFF: Heap (16 MB)
0x8000000+:           Identity-mapped (128 MB total)
0xFEC00000:           IOAPIC MMIO
0xFEE00000:           LAPIC MMIO
```

## Key Volatile Addresses

```
0x600000:  Process table (256 bytes per PID, 16 PIDs)
0x6F800:   Command buffer (64 bytes max)
0x6FA00:   VGA cursor row
0x6FA08:   VGA cursor col
0x6FBE0:   Shift key state
0x6FBE8:   Caps lock state
0x6FBF0:   Keyboard ring buffer read index
0x6FBF8:   Keyboard ring buffer write index
0x6FC00:   Keyboard ring buffer (128 bytes)
0x6FC04:   TSS.RSP0
0x6FD00:   Command history (circular buffer)
0x6FE08:   Current PID
0x6FF00:   Multiboot2 info pointer
```

## Related Repos

- **Compiler:** `/home/primecore/Documents/Fajar Lang` (fajar-lang)
- **ARM64 FajarOS:** `github.com/fajarkraton/fajar-os`
- **Monolithic source:** `fajar-lang/examples/fajaros_nova_kernel.fj` (9,019 LOC original)

## Key Differences from FajarOS Surya (ARM64)

| Aspect | Nova (x86_64) | Surya (ARM64) |
|--------|---------------|---------------|
| Boot | Multiboot2 / GRUB | UEFI (QCS6490) |
| Privilege | Ring 0/3 | EL0/EL1 |
| Paging | CR3 -> PML4 | TTBR0/TTBR1 |
| Interrupts | IDT + LAPIC | VBAR_EL1 + GICv3 |
| Syscalls | SYSCALL/SYSRET | SVC |
| Timer | PIT + LAPIC | Architected timer |
| Serial | 16550 I/O port | PL011 MMIO |

## Session Protocol

1. Read this CLAUDE.md (auto-loaded)
2. Read `docs/PLAN.md` for current sprint
3. Build: `make build`
4. Test: `make run` or `make run-kvm`
5. When adding code, edit the specific module file (NOT build/combined.fj)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fajarkraton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
