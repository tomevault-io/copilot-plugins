## zzusage

> Zig 0.15 reimplementation of ccusage (Claude Code usage analyzer).

# zzusage Agent Notes

Zig 0.15 reimplementation of ccusage (Claude Code usage analyzer).

## Build

```bash
zig build        # compile
zig build run    # run
zig build test   # test
```

## Key References

- [Zig 0.15 I/O API](docs/zig-0.15-io-api.md) - stdout, writers, ArrayList changes
- libvaxis dependency for TUI mode

## Architecture

- `src/main.zig` - single-file implementation
- JSONL parsing from `~/.config/claude/projects/` and `~/.claude/projects/`
- Commands: daily, weekly, monthly, sessions, blocks, statusline

---
> Source: [joelreymont/zzusage](https://github.com/joelreymont/zzusage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
