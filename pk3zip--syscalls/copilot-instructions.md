## syscalls

> - Core C sources live at the repo root (`main.c`, `lib.c`, `malloc.c`) with headers alongside. Keep public interfaces in `*.h` and implementations in matching `*.c`/`*.cpp`.

# Repository Guidelines

## Project Structure & Module Organization
- Core C sources live at the repo root (`main.c`, `lib.c`, `malloc.c`) with headers alongside. Keep public interfaces in `*.h` and implementations in matching `*.c`/`*.cpp`.
- Assembly entry points and syscall shims are in `entry.asm`, `syscalls.asm`, and `utils.asm`. Prefer adding new low-level routines there rather than inlined asm.
- The C++ launcher is `writer.cpp` calling `c_main` from `main.h`.
- Build artifacts collect under `build/`, while the linked outputs `syscalls` (ELF) and `syscalls.bin` stay in the root. Avoid checking in anything under `build/` or `*.bin`.

## Build, Test, and Development Commands
- `make`: compile C, C++, and NASM sources into `build/`, link `syscalls`, and emit `syscalls.bin` via `objcopy`.
- `make clean`: remove `build/`, `syscalls`, and `syscalls.bin`.
- `./syscalls`: run the built ELF to sanity-check argv/env printing and hostname readout.
- Toolchain expectations: `gcc`, `g++`, `nasm`, and `ld` available for x86_64 Linux; no external libs beyond glibc headers used for type definitions.

## Coding Style & Naming Conventions
- C/C++: use 4-space indentation, no tabs; keep functions small and syscall-oriented. Avoid pulling in the standard library at runtime—use the syscall wrappers in `syscalls.h` and helpers in `lib.c`.
- Assembly: NASM syntax with `default rel`; keep labels lowercase with underscores.
- Names: match header/impl pairs (`foo.h`/`foo.c`), keep globals prefixed clearly (e.g., `sys_`, `mem_`), and avoid anonymous magic numbers—prefer small `#define`s near use.

## Testing Guidelines
- No formal test harness is present; lean on `./syscalls` for manual sanity checks.
- When adding functionality, create minimal repro code in `main.c` or a temporary helper under `build/` to validate new syscalls or allocator changes before integrating.
- If adding scripts or tests, keep them self-contained and runnable via `make test`-style phony targets.

## Commit & Pull Request Guidelines
- Commit messages follow short, imperative phrasing seen in history (e.g., `add default rel`, `change main to return`). Keep scope narrow and commits small.
- Pull requests should state intent, list key changes, and mention how you validated (commands run, manual checks). Link any relevant issues and note ABI or syscall-surface changes explicitly.
- Include screenshots or logs only when they clarify runtime behavior (e.g., sample `./syscalls` output after new features).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pk3zip)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pk3zip)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
