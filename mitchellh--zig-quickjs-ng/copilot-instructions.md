## zig-quickjs-ng

> A file for [guiding coding agents](https://agents.md/).

# Agent Development Guide

A file for [guiding coding agents](https://agents.md/).

- The C API is defined in `quickjs.h`
- `quickjs.h` is in `.zig-cache`. If it isn't there, run `zig build` once.
- Write comprehensive unit tests for all new APIs.
- Mimic the style of the existing Zig codebase.
- Examples have their own `build.zig`, so run `zig` commands for
  examples from within their respective directories.

## Zig Guide

- Use Zig types in API parameters where possible.
- Never use `[*:0]const u8` in public APIs; use `[:0]const u8` and
  call `ptr` on it instead.
- For definitions, use `context x: T = .` syntax, do NOT use
  `const x = T{}`. This works for functions too: `const x: T = .init()`
- For packed structs or enums porting the C API, always pair it
  with unit tests to ensure we match C constants
- For types that should match C layouts exactly (extern structs,
  enums, packed structs, etc.), always add a comptime assertion
  that the size and alignment matches the header
- For callbacks, use the `opaque.zig` helpers and make the callback
  use free (no conversion cost) Zig types wherever possible. See
  Runtime.addFinalizer or setPromiseHook for an example.
- Try to restrict line length to ~80, using trailing commas for
  function args or structs where possible to split them up. Don't
  do this for short lines that fit easily.
- Don't import other files within a test, use a top-level import.

## Testing Guide

- Tests should use real JavaScript wherever possible (use `eval`)
  and avoid simply twiddling internal state.
- When modifying the core library, also verify all examples
  build. Use subagents for this.

---
> Source: [mitchellh/zig-quickjs-ng](https://github.com/mitchellh/zig-quickjs-ng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
