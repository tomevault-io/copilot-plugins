## nvim

> This is a LazyVim-based Neovim configuration for Neovim 0.12+, migrated from LunarVim.

# AGENTS.md — Neovim Config

This is a LazyVim-based Neovim configuration for Neovim 0.12+, migrated from LunarVim.

## Structure

```
~/.config/nvim/
├── init.lua                      # Entry point, loads config/lazy.lua
├── lua/config/
│   ├── lazy.lua                  # lazy.nvim bootstrap + LazyVim extras imports
│   ├── options.lua               # Custom vim options
│   ├── keymaps.lua               # Custom keymaps
│   └── autocmds.lua              # Custom autocommands
└── lua/plugins/
    ├── colorscheme.lua           # NeoSolarized theme
    ├── python.lua                # basedpyright + uv .venv detection
    └── ui.lua                    # Smooth scroll, breadcrumbs (nvim-navic)
```

## Key conventions

- **Extras** (language packs, formatting, editor features) are imported in `lua/config/lazy.lua` inside the `spec` table, between the LazyVim import and `{ import = "plugins" }`.
- **Custom plugin specs** go in `lua/plugins/*.lua` as individual files. Each file returns a table of lazy.nvim plugin specs.
- **Do not** put `lazyvim.plugins.extras.*` imports inside `lua/plugins/` files — they must be in the `spec` table in `lazy.lua` to load in the correct order.
- `lazy-lock.json` is tracked in git to pin plugin versions.
- `lazyvim.json` is gitignored (auto-generated state).

## Enabled extras

- `formatting.prettier` — JS/TS/MD formatting
- `lang.go` — gopls, gofumpt, goimports, golangci-lint
- `lang.json` — JSON schemas via SchemaStore
- `lang.yaml` — YAML LSP
- `lang.docker` — Dockerfile LSP
- `lang.markdown` — Markdown preview and rendering
- `ui.indent-blankline` — Indent guides
- `editor.illuminate` — Word highlighting under cursor

## Python setup

basedpyright is configured in `lua/plugins/python.lua` to auto-detect:
1. `.venv/bin/python` in the project root (uv, poetry)
2. `$VIRTUAL_ENV/bin/python` as fallback

No venv activation needed — just open nvim in a project with `.venv/`.

## LSP servers (via Mason)

basedpyright, gopls, lua-language-server, golangci-lint, gofumpt, goimports, prettier, shfmt, stylua

## Adding a new language

1. Check available extras: https://www.lazyvim.org/extras
2. Add `{ import = "lazyvim.plugins.extras.lang.<name>" }` to the spec in `lua/config/lazy.lua`
3. Run `:Lazy sync` to install
4. Mason will auto-install the required LSP server

## Migration notes

Migrated from LunarVim (April 2025) due to LunarVim being effectively unmaintained and incompatible with Neovim 0.12. The old LunarVim config is backed up at `~/.config/lvim.bak/` and runtime at `~/.local/share/lunarvim/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kavinjsir)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kavinjsir)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
