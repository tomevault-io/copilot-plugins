## p-nvim-config

> Dotfiles for a Python/web development Neovim setup with Catppuccin theme.

# CLAUDE.md — Neovim Config

Dotfiles for a Python/web development Neovim setup with Catppuccin theme.

## Quick Facts

| Aspect         | Detail                                        |
| -------------- | --------------------------------------------- |
| Plugin manager | lazy.nvim                                     |
| Theme          | Catppuccin (auto: latte/mocha)                |
| Leader key     | `<Space>`                                     |
| Background     | auto (terminal-dependent)                     |
| Indent         | Python: 4 spaces, others: 2                   |
| LSP servers    | pyright (Python), ruff (Python), html, djlint |
| Formatters     | conform.nvim (stylua, prettier, djlint, ruff, sqlfluff) |
| Debug          | nvim-dap + debugpy (Python)                   |
| DB             | vim-dadbod + dadbod-ui                        |
| HTTP Client    | kulala.nvim                                   |
| AI Assistant   | sidekick.nvim                                 |
| Diff Viewer    | diffview.nvim                                 |
| Icon Provider  | mini.icons                                    |
| Commenting     | Comment.nvim (gc/gb)                          |

## Project Structure

```
%LOCALAPPDATA%\nvim\          # Windows config (this repo)
├── init.lua                  # Entry point — bootstraps lazy.nvim, loads vim-options + plugins
├── lua/
│   ├── vim-options.lua       # Core options, leader key, window nav keymaps, filetype overrides
│   ├── plugins.lua           # Empty stub (return {}) — kept for lazy.nvim discovery
│   └── plugins/              # One file per plugin or plugin group
│       ├── comment.lua       # Comment toggle (Comment.nvim, gc/gb)
│       ├── alpha.lua         # Start screen (ASCII art dashboard)
│       ├── aerial.lua        # Code outline tree
│       ├── catppuccin.lua    # Theme setup (auto light/dark)
│       ├── completions.lua   # nvim-cmp + LuaSnip + sources
│       ├── debugging.lua     # nvim-dap + dap-ui + dap-python
│       ├── git-stuff.lua     # vim-fugitive + diffview.nvim + gitsigns
│       ├── lualine.lua       # Status line (with venv display)
│       ├── lsp-config.lua    # Mason + lspconfig (ruff, html, djlint)
│       ├── mini-icons.lua    # Icons (mini.icons)
│       ├── neo-tree.lua      # File explorer + bufferline
│       ├── conform.lua       # Formatters (stylua, prettier, djlint, ruff, sqlfluff)
│       ├── oil.lua           # Directory editing
│       ├── sidekick.lua      # AI assistant (CLI)
│       ├── snacks.lua        # Animations, scroll, indent, LazyGit
│       ├── telescope.lua     # Fuzzy finder (fzf-native + ui-select)
│       ├── treesitter.lua    # Syntax highlighting + indent (uses MSVC 'cl' on Windows)
│       ├── venv-selector.lua # Python venv auto-selection
│       ├── vim-dadbod.lua    # Database UI
│       ├── which-key.lua     # Key binding popup hints
│       ├── kulala.lua        # HTTP client (.http, .rest)
│       └── neotest.lua       # Tests (pytest, nvim-neotest)
└── .gitignore
```

## Plugin Management

Manager: [lazy.nvim](https://github.com/folke/lazy.nvim).

### Adding a New Plugin

1. Create `lua/plugins/<name>.lua`
2. Return a lazy.nvim spec table:

```lua
return {
    "owner/repo",
    lazy = false,
    config = function()
        require("plugin").setup({})
    end,
    keys = { { "<leader>x", "<cmd>PluginCmd<cr>", desc = "Plugin command" } },
    event = "VeryLazy",
}
```

### Style Conventions

- **4-space indentation** everywhere
- Prefer explicit `config = function()` over `opts = {}` when keymaps are involved
- Always include `desc = "..."` on keymaps (shows in which-key)
- Group related keymaps with comments
- One logical plugin (or tightly coupled group) per file

## LSP Configuration

Stack: `mason.nvim` → `mason-lspconfig.nvim` → `nvim-lspconfig`

Current servers (in `lua/plugins/lsp-config.lua`):

- **ruff** — Python linting/formatting
- **html** — HTML language support
- **djlint** — HTML template formatting
- **pyright** — Python type checking

LSP keymaps (active only on LspAttach):

| Key          | Action              |
| ------------ | ------------------- |
| `K`          | Hover documentation |
| `<leader>ld` | Go to definition    |
| `<leader>lr` | Find references     |
| `<leader>la` | Code action         |

## Formatting (conform.nvim)

Defined in `lua/plugins/conform.lua`:

| Formatter              | Language                  |
| ---------------------- | ------------------------- |
| ruff_fix + ruff_format | Python                    |
| stylua                 | Lua                       |
| prettier               | JS/JSON/CSS/Markdown/YAML |
| djlint                 | HTML templates            |
| sqlfluff               | SQL (PostgreSQL)          |

Trigger: `<leader>gf` → `conform.format({ lsp_fallback = true })`

Format on save is enabled automatically.

## Keymaps Overview

### Window Navigation (vim-options.lua)

| Key     | Action       |
| ------- | ------------ |
| `<C-h>` | Window left  |
| `<C-j>` | Window down  |
| `<C-k>` | Window up    |
| `<C-l>` | Window right |

### General

| Key         | Action                 |
| ----------- | ---------------------- |
| `<leader>h` | Clear search highlight |

### Terminal (vim-options.lua)

| Key          | Action                         |
| ------------ | ------------------------------ |
| `<Esc>`      | Exit terminal to normal mode   |
| `<leader>pt` | Open PowerShell terminal       |

### Telescope (telescope.lua)

| Key                | Action     |
| ------------------ | ---------- |
| `<C-p>`            | Find files |
| `<leader>fg`       | Live grep  |
| `<leader><leader>` | Old files  |

### Neo-tree (neo-tree.lua)

| Key          | Action                |
| ------------ | --------------------- |
| `<C-n>`      | Toggle file explorer  |
| `<leader>bf` | Open buffers in float |

### Oil (oil.lua)

| Key | Action                         |
| --- | ------------------------------ |
| `-` | Toggle directory edit in float |

### Git (git-stuff.lua)

| Key          | Action                    |
| ------------ | ------------------------- |
| `<leader>gh` | Preview hunk              |
| `<leader>gb` | Toggle current line blame |
| `<leader>gd` | Show diff (gitsigns)      |
| `<leader>gD` | Open diff view (diffview) |
| `<leader>gx` | Close diffview            |
| `<leader>gn` | Next hunk                 |
| `<leader>gN` | Prev hunk                 |
| `<leader>ga` | Stage hunk                |
| `<leader>gu` | Undo stage hunk           |
| `<leader>gA` | Add file in stage         |
| `<leader>gs` | LazyGit (via Snacks)      |

### Debug (debugging.lua)

| Key          | Action            |
| ------------ | ----------------- |
| `<Leader>db` | Toggle breakpoint |
| `<Leader>dc` | Continue          |
| `<Leader>dt` | Toggle DAP UI     |

### Aerial (aerial.lua)

| Key         | Action              |
| ----------- | ------------------- |
| `<leader>o` | Toggle code outline |

### Sidekick (sidekick.lua)

| Key          | Action                       |
| ------------ | ---------------------------- |
| `<leader>ko` | Toggle AI CLI                |
| `<leader>kc` | Close AI CLI                 |
| `<C-_>`      | Focus AI CLI from any mode   |

### HTTP Client (kulala.lua)

| Key          | Action            |
| ------------ | ----------------- |
| `<leader>Rs` | Send request      |
| `<leader>Ra` | Send all requests |
| `<leader>Rb` | Open scratchpad   |

Active in `.http` and `.rest` files.

### Tests (neotest.lua)

| Key          | Action                 |
| ------------ | ---------------------- |
| `<leader>tn` | Run nearest test       |
| `<leader>tf` | Run current file tests |
| `<leader>ta` | Run test suite         |
| `<leader>tl` | Run last test          |
| `<leader>tt` | Test summary tree      |
| `<leader>to` | Test output            |
| `<leader>td` | Debug nearest test     |
| `<leader>ts` | Test output panel      |

### Completions (completions.lua)

| Key         | Action             |
| ----------- | ------------------ |
| `<C-Space>` | Trigger completion |
| `<CR>`      | Confirm selection  |
| `<C-e>`     | Abort              |
| `<C-b>`     | Scroll docs up     |
| `<C-f>`     | Scroll docs down   |

### Comments (comment.lua)

Plugin: [numToStr/Comment.nvim](https://github.com/numToStr/Comment.nvim) — standard gc/gb mappings

| Key     | Action                            |
| ------- | --------------------------------- |
| `gcc`   | Toggle line comment (current line)|
| `gcNc`  | Toggle line comment (N lines)     |
| `gbc`   | Toggle block comment (current)    |
| `gbNc`  | Toggle block comment (N lines)    |
| `gc` + m| Operator mode (e.g. `gcw`, `gc$`) |
| `gb` + m| Block operator mode               |
| `gc` (v)| Visual mode: toggle on selection  |

All comments are toggleable — press again to uncomment.

## Adding Language Support

1. Add LSP server in `lua/plugins/lsp-config.lua` using `vim.lsp.config()`
2. Treesitter parsers install automatically (`auto_install = true`)
3. Add formatters/linters to `lua/plugins/conform.lua` if needed

## Disabled Plugins (backed up)

Files with `.back` extension — currently disabled:

- `nvim-tmux-navigation.back` — Tmux nav

To re-enable: remove `.back` extension.

## Common Tasks

```
:Lazy              # Plugin status dashboard
:Lazy update       # Update all plugins
:Lazy sync         # Sync after config changes
:TSUpdate          # Update Treesitter parsers
:Mason             # Mason UI (install/manage LSPs)
:Dblast            # Open Dadbod UI (databases)
```

## Notes

- `vim-options.lua` uses `vim.cmd("set ...")` style — matches existing convention
- Swap files disabled (`vim.opt.swapfile = false`)
- Background is auto-detected from terminal — Catppuccin latte/mocha
- `.venv` auto-detected by venv-selector (Python projects)
- Dynamic indentation: Python — 4 spaces, JS/TS/Lua/HTML/CSS/SQL/JSON/YAML/Markdown — 2 spaces
- Filetype overrides for .rest -> http are defined in vim-options.lua

---
> Source: [Petro-lium/p_nvim_config](https://github.com/Petro-lium/p_nvim_config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
