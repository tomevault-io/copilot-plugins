## zigtwitarchiveutil

> AGENTS.md — zigtwitarchiveutil (Zig 0.12+)

AGENTS.md — zigtwitarchiveutil (Zig 0.12+)
- Build: `zig build` (installs into `zig-out/`)
- Run: `zig build run -- <args>`
- Test (all via build): `zig build test`
- Test (single file): `zig test src/root.zig`
- Test (single test): `zig test src/root.zig --test-filter "Parse test"`
- Format (fix): `zig fmt .`
- Format (check CI): `zig fmt --check .`
- Lint: use compiler warnings; optional `zig ast-check src/*.zig`
- Imports: `@import("std")`; library as `@import("zigtwitarchiveutil")`; re-export in `src/root.zig`.
- Types: structs/enums/unions/error sets TitleCase; errors like `error.ParseError`.
- Functions, variables, fields: lowerCamelCase; concise, descriptive names.
- Files/modules: lower_snake_case; public API routed through `src/root.zig`.
- Errors: prefer `!T` error unions; bubble with `try`; only `catch` to handle.
- Memory: pass `std.mem.Allocator`; use `defer`/`errdefer`; free owned memory.
- Testing: co-locate `test` blocks; use `std.testing` and `testing.allocator`; keep tests hermetic.
- Logging: `std.debug.print` for local dev; keep output minimal in library code.
- Docs: use `///` for items and `//!` at file tops.
- Cursor/Copilot rules: none found in repo; follow them if added later.
- Commit scope: small, focused changes; explain “why” in messages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/justinhj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/justinhj)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
