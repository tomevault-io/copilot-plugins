## lazyissues

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`lazyissues` is a pure-Lua Neovim plugin (Neovim 0.10+) — a lazygit-style TUI for a file-based issue tracker whose data lives as `issue.json` files under an `Issues/` folder. No build step, no package manager. Runtime deps: `MunifTanjim/nui.nvim` (UI), and `git` (optional, for branch-edit markers).

## The core constraint: byte-identical JSON

The JSON codec (`lua/lazyissues/json.lua`) must serialize `issue.json` files **byte-for-byte identically** to the canonical .NET `System.Text.Json` output, so edits produce minimal git diffs. When touching the codec, schema, or anything that writes files, preserve:

- **PascalCase** field names; **enums as strings** (not numbers); **null** values kept.
- All non-ASCII / special chars escaped to `\uXXXX` (mimics .NET `JavaScriptEncoder`).
- 2-space indent.
- Timestamps: UTC ISO 8601 with **7 fractional digits**, e.g. `2026-06-27T12:00:00.0000000Z` (.NET `DateTime` "O" format).
- UUIDs: RFC 4122 v4, generated client-side.

The field schema, enum values, and defaults in `lua/lazyissues/config.lua` are **hardcoded to mirror the .NET backend model** — changing them requires keeping both sides in sync.

## Verify changes

Format and the roundtrip codec test are the two checks; final verification is manual in Neovim (open the plugin and exercise the change).

- **Format (run after editing any `.lua` file):** `stylua .` — config in `stylua.toml` (100 cols, 2-space, prefer double quotes, always call parens). (A PostToolUse hook also runs `stylua` on edited `.lua` files automatically once stylua is installed.)
- **Lint:** `luacheck lua/ test/` — config in `.luacheckrc` (Neovim-tuned). Don't introduce new warnings; 0 errors expected.
- **Roundtrip fidelity test** (run when touching the codec, schema, or store): see the `/roundtrip` skill, or run from the repo root:
  ```
  nvim --headless --clean \
    -c "lua package.path='./lua/?.lua;./lua/?/init.lua;./?.lua;'..package.path" \
    -c "lua require('test.roundtrip').run()" -c "qa!"
  ```
  It decodes then re-encodes every `issue.json` and reports byte-identical vs diffs. Runs against the bundled fixture by default; pass a data-root path to `run()` to test real data.

## Git workflow

Always work on a feature branch and open a PR — **never commit directly to `main`**. Match the existing commit style: lowercase, imperative, concise (e.g. "Fix template picker border visibility").

---
> Source: [Interlude-Software/lazyissues](https://github.com/Interlude-Software/lazyissues) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
