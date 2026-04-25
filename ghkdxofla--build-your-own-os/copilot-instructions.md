## build-your-own-os

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust implementation of a simple operating system based on the "Operating System in 1,000 Lines" tutorial, targeting RISC-V architecture. The project maintains two parallel implementations:

- **Auto-generated version** (`src/`) - Complete AI-generated implementation
- **Manual implementation** (`src-manual/`) - Template for manual development

## Build Commands

```bash
# Build auto-generated version
cargo build --release --features=auto

# Build manual implementation version
cargo build --release --features=manual

# Run tests (includes build verification and output comparison)
cargo test --test helper_script -- --nocapture
cargo test --test implementation_comparison
```

## Running the OS

```bash
# Run with QEMU (adjust path as needed)
qemu-system-riscv64 -machine virt -bios none -kernel target/riscv64gc-unknown-none-elf/release/rust-os-1000-lines -nographic

# Exit QEMU: Ctrl+A, then X
```

## Architecture Overview

**Boot Process**: Assembly code in `boot.S` sets up the RISC-V environment, configures stack pointer, handles multi-hart synchronization, then jumps to Rust main at entry point `0x80200000`.

**Core Components**:
- **UART Driver** (`uart.rs`): 16550-compatible UART at `0x1000_0000` for console I/O
- **Memory Manager** (`memory.rs`): Bump allocator with 4KB pages, provides kmalloc/kfree
- **Process Scheduler** (`process.rs`): Round-robin scheduling with context switching
- **File System** (`fs.rs`): Simple inode-based filesystem with directory entries

**Build System**: Custom linker script (`linker.ld`), build.rs handles assembly compilation, feature flags switch between implementations.

## Prerequisites

Required tools and setup:
```bash
# Add RISC-V target
rustup target add riscv64gc-unknown-none-elf

# Install required components  
rustup component add rust-src

# Ensure QEMU with RISC-V support is installed
```

## Development Notes

- Target: `riscv64gc-unknown-none-elf` (no_std environment)
- Both implementations should produce identical functionality
- Console prompts differ: `rust-os>` (auto) vs `rust-os-manual>` (manual)
- Tests verify build success and compare outputs between implementations
- OpenSBI firmware binary included in `script/` directory for RISC-V32 execution

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ghkdxofla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
