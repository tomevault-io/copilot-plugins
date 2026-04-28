## nvim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup

Install by symlinking this directory to `~/.config/nvim`:

```sh
./install.sh
```

This backs up any existing `~/.config/nvim` and creates a symlink. Requires Neovim 0.12+.

## Architecture

Single-file configuration based on kickstart.nvim. Everything — options, keymaps, autocommands, plugin declarations, and plugin configs — lives in `init.lua`.

**Plugin management:** Neovim 0.12's built-in `vim.pack`. Plugins are declared in one `vim.pack.add({...}, { load = true })` call. Lockfile: `nvim-pack-lock.json`. Update with `:lua vim.pack.update()`.

**`after/ftplugin/markdown.lua`** — sets `conceallevel=2` for obsidian.

## Key plugin decisions

- **LSP:** `nvim-lspconfig` + `mason.nvim` + `mason-lspconfig`. Configured servers: `pyright`, `lua_ls`. Formatter: `stylua` (via mason-tool-installer).
- **Completion:** `blink.cmp` with LuaSnip snippets; `lazydev.nvim` for Neovim Lua API completion.
- **Formatting:** `conform.nvim` — formats on save for Lua (`stylua`), LSP fallback for other filetypes. Manual: `<leader>f`.
- **Treesitter:** Native Neovim 0.12 engine. Bundled parsers: c, lua, markdown, markdown_inline, query, vim, vimdoc. Install extras with `:TSInstall <lang>`.
- **Fuzzy finding:** Telescope with fzf-native and ui-select extensions.
- **Markdown:** render-markdown.nvim + markdown-preview.nvim + obsidian.nvim — all three loaded together in a single `FileType markdown` autocommand. Obsidian workspaces point to iCloud vaults.
- **LaTeX:** vimtex with latexmk, output in `build/`. Globals must be set before plugin loads (already handled in `init.lua`).
- **File explorer:** oil.nvim as default explorer. `<leader>e` opens oil; `<leader>o` inside oil opens the file under cursor with the system app (e.g. Preview for PDFs).
- **Git:** gitsigns for gutter signs (no keymaps configured beyond defaults).
- **Highlight colors:** Set after `vim.cmd.colorscheme` to avoid being overwritten. Currently `GitSignsAdd` is set to `#9ece6a`.

## Adding a new plugin

Add a `vim.pack` entry to the list in `init.lua` and configure it inline in the same file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jcconnects) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
