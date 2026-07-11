## lazyvim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Neovim configuration built on **LazyVim** with **lazy.nvim** as the plugin manager. Entry point is `init.lua` -> `lua/config/lazy.lua`.

## Architecture

- `lua/config/` — Core settings loaded automatically by LazyVim:
  - `lazy.lua` — Bootstraps lazy.nvim and loads LazyVim + custom plugin specs
  - `options.lua` — Vim options and LazyVim globals
  - `keymaps.lua` — Custom keybindings (buffer cycling, text bubbling, Go struct tags, markdown formatting)
  - `autocmds.lua` — Custom autocommands (e.g., disable autoformat for `.env` files)
- `lua/plugins/` — Plugin specs (one per file), merged with LazyVim defaults by lazy.nvim

## Language Support

Configured via LazyVim extras (`lazyvim.json`) and custom plugin specs:

| Language | LSP | Formatter/Linter |
|----------|-----|-------------------|
| Python | Pyright (with venv detection) | Ruff |
| Ruby | ruby_lsp (via mise shims) | Rubocop |
| Go | gopls (gofumpt, staticcheck) | gopher.nvim for struct tags |
| TypeScript | tsserver | LazyVim default |
| Lua | lua_ls | stylua |

Tool versions are managed by **mise** — shims are prepended to PATH in `options.lua`.

## Key Customizations from LazyVim Defaults

- `vim.g.root_spec = { "cwd" }` — project root is always cwd, not auto-detected git root
- `vim.o.exrc = true` — project-local `.nvim.lua` / `.exrc` files are loaded
- Comment toggle remapped to `\\` (from `gc`) in `mini.comment.lua`
- Relative line numbers disabled
- Animations disabled (`snacks_animate = false`)
- DAP (debug adapter protocol) enabled for Go, Python, Ruby

## Formatting

- Lua files: **stylua** (2-space indent, 120 column width — see `stylua.toml`)
- Format-on-save is enabled globally via LazyVim's `vim.g.autoformat = true`
- `.env` files are excluded from autoformat via autocmd

---
> Source: [jcarley/lazyvim](https://github.com/jcarley/lazyvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
