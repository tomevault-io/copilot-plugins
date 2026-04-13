## lean4-risczero

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RISC Zero zkVM sample demonstrating C FFI integration with Lean4-to-C transpilation. The guest program calls C functions (including Lean4-generated code) via FFI, and the host generates and verifies zero-knowledge proofs.

## Build and Run Commands

```sh
# Run the prover (debug build)
cargo run -p host

# Run with optimizations (faster proving)
cargo run -p host --release

# Build without running
cargo build -p host

# Check compilation
cargo check
```

### Lean4 Commands

```sh
# Build Lean4 and generate C code
cd lean4-fib && lake build +Main:c

# Generated C output location
# .lake/build/ir/Main.c
```

## Architecture

```
risczero-ffi/
├── host/src/main.rs      # Prover: sends input, generates proof, verifies receipt
├── methods/
│   ├── build.rs          # Embeds guest ELF via risc0_build::embed_methods()
│   ├── src/lib.rs        # Exports GUEST_ELF and GUEST_ID constants
│   └── guest/
│       ├── build.rs      # Cross-compiles C code to RISC-V using cc crate
│       ├── src/main.rs   # Guest: reads input, calls C FFI, commits result
│       └── src/c_code/   # C source files (manual and Lean4-generated)
└── lean4-fib/ # Lean4 project for C code generation
    ├── Main.lean         # Lean4 source with @[export] functions
    └── lakefile.toml
```

### Data Flow

1. Host writes input (`n`) to `ExecutorEnv`
2. Guest reads input via `env::read()`
3. Guest calls C function via FFI (`lean_fibonacci`)
4. Guest commits result via `env::commit()`
5. Host extracts result from `receipt.journal.decode()`
6. Host verifies proof with `receipt.verify(GUEST_ID)`

### C FFI Build Process

The guest's `build.rs` uses the RISC Zero RISC-V GCC toolchain located at `~/.risc0/cpp/bin/riscv32-unknown-elf-gcc`. C files are compiled with:
- Target: `rv32im` (RISC-V 32-bit with M extension)
- ABI: `ilp32`
- Flags: `-ffreestanding -nostdlib`

## Lean4 to RISC Zero Pipeline

```
Lean4 (UInt32) → lake build +Main:c → C code → RISC-V GCC → RISC Zero Guest → Proof
```

### Lean4 Constraints for zkVM

When writing Lean4 code for RISC Zero:
- **Use only**: `UInt32`, `UInt64`, `Bool` (primitive types)
- **Avoid**: `Nat`, `Array`, `String` (require Lean runtime, not available in zkVM)
- **Recursion**: Use `partial` or provide termination proofs
- **Export**: Use `@[export function_name]` attribute to generate C functions

### Extracting Pure C from Lean4

The generated `.lake/build/ir/Main.c` contains Lean runtime calls. For zkVM, manually extract the pure computation logic into `methods/guest/src/c_code/lean_fibonacci.c`.

## Prerequisites

- Rust toolchain
- RISC Zero toolchain: `cargo install cargo-risczero && cargo risczero install`
- Lean4 (for modifying Lean code): Install via elan

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adust09)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adust09)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
