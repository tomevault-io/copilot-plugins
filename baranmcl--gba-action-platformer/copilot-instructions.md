## gba-action-platformer

> - **ROM:** `bash tools/build_rom.sh` (canonical — regenerates level headers from tools/levels/, then builds via devkitPro). Plain `make` builds against whatever generated headers are committed — run `make levels` first if you use it.

# Spronk Quest — Project Guidance

## Build

- **ROM:** `bash tools/build_rom.sh` (canonical — regenerates level headers from tools/levels/, then builds via devkitPro). Plain `make` builds against whatever generated headers are committed — run `make levels` first if you use it.
- **Host tests:** run `bash tools/host_test.sh` (canonical — regenerates level headers, then compiles + runs). Plain `make -C test` is fragile on this machine; see CLAUDE.md.

## Architecture: Three-Layer Rule

```
src/logic/   + include/logic/   — pure C++17, NO bn:: types (host-unit-testable)
src/engine/                     — Butano glue (bn:: allowed here only)
src/game/                       — scenes, entry point
```

**Never** put `bn::` types under `include/logic/` or `src/logic/`. This breaks host tests.

## Before Committing

Run the purity guard to catch accidental Butano leakage into the logic layer:

```
python tools/check_logic_purity.py
```

## References

- `docs/pitfalls/implementation-pitfalls.md` — common implementation traps
- `docs/pitfalls/testing-pitfalls.md` — common testing traps

---
> Source: [baranmcl/GBA-action-platformer](https://github.com/baranmcl/GBA-action-platformer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
