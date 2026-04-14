## ocaml64

> OCaml64 is a cycle-accurate Commodore 64 (C64) emulator written in OCaml. It prioritizes hardware fidelity by simulating the system at the clock-cycle level rather than the instruction level.

# GEMINI.md - OCaml64 Project Context

## Overview
OCaml64 is a cycle-accurate Commodore 64 (C64) emulator written in OCaml. It prioritizes hardware fidelity by simulating the system at the clock-cycle level rather than the instruction level.

## Architecture
- **CPU (`lib/c6510.ml`)**: Implements the MOS 6510. Features a cycle-by-cycle `tick` function and a full 6502/6510 instruction decoder.
- **CIA (`lib/cia.ml`)**: Emulates MOS 6526 Complex Interface Adapters, including Timers (A/B), TOD clocks, and I/O port logic.
- **System Integration (`lib/c64.ml`)**: Defines the `Computer` module.
    - **MMU**: Handles C64 bank-switching ($00/$01 CPU ports) for RAM, ROM (Kernal, BASIC, CharGen), and I/O mapping.
    - **Bus**: Manages data/address lines between components.
- **TUI/CLI (`bin/main.ml`)**:
    - Uses the `Mosaic` library for a terminal-based debugger.
    - Includes a Zstandard-compressed trace recording system for execution replay.

## Tech Stack
- **Language**: OCaml (Targeting modern versions with Multicore support/`Domainslib`).
- **Standard Library**: `Base` and `Stdio`.
- **Build System**: Dune.
- **Testing**: `ppx_expect` for inline hardware verification tests.
- **UI**: `Mosaic` for TUI components.
- **Compression**: `Zstandard` for trace files.

## Development Mandates
1. **Cycle Accuracy**: All hardware logic must be implemented within `tick` or `fetch_decode_execute` loops. Do not implement "macro" instructions that bypass the cycle-by-cycle state machine.
2. **State Immutability**: Prefer returning new state records (e.g., `Some { cpu with ... }`) to maintain the functional simulation of hardware transitions.
3. **Verification**: Every new instruction or hardware feature MUST include an expect test in `lib/c64.ml` to verify register and flag state against known C64 behavior.
4. **Memory Access**: Always use the `MMU` within `Mem` to access memory to ensure bank-switching logic is respected.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gloppasglop)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gloppasglop)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
