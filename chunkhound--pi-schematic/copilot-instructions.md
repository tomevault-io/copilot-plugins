## pi-schematic

> **Never use `console.debug/warn/error/log`** — writes to stdout/stderr corrupt pi's TUI ANSI rendering. Extension host runs in the same process.

# TUI Safety

**Never use `console.debug/warn/error/log`** — writes to stdout/stderr corrupt pi's TUI ANSI rendering. Extension host runs in the same process.

Use `ctx.ui.notify()` / `setStatus()` / `setWidget()` instead. For diagnostics, remove entirely.

---
> Source: [chunkhound/pi-schematic](https://github.com/chunkhound/pi-schematic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
