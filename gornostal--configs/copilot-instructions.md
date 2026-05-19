## configs

> This project contains configs for apps that run via CLI.

# My configs

This project contains configs for apps that run via CLI.

## Instructions

When doing any significant modifications, change this file also.

## Activation

Run `setup.sh` to activate all configs. It symlinks the Neovim, Tmux, Fish, and Claude Code configs into their standard locations, skipping any that already exist.

```bash
./setup.sh
```


## Neovim Configuration

```
nvim/
├── init.lua              # Entry point
├── lua/
│   ├── config/
│   │   ├── options.lua   # Basic settings (line numbers, tabs, etc.)
│   │   └── keymaps.lua   # Key bindings
│   └── plugins/
│       └── init.lua      # All plugins (LSP, completion, theme)
└── .gitignore
```

On first launch of Neovim, lazy.nvim will auto-install all plugins. Then Mason will install the language servers (pyright for Python, omnisharp for .NET).

## Plugins included

| Plugin | Description |
|--------|-------------|
| **lazy.nvim** | Plugin manager |
| **catppuccin** | Color scheme (mocha flavor) |
| **copilot.vim** | GitHub Copilot AI code completion |
| **neoscroll.nvim** | Smooth scrolling animations |
| **telescope.nvim** | Fuzzy finder for files, grep, buffers |
| **flash.nvim** | Quick jumps with labels |
| **gitsigns.nvim** | Git change indicators in the gutter |
| **mason.nvim** | Language server installer |
| **mason-lspconfig.nvim** | Auto-configures LSP servers from Mason |
| **nvim-lspconfig** | LSP configuration |
| **nvim-cmp** | Autocompletion engine |
| **LuaSnip** | Snippet engine |
| **nvim-treesitter** | Syntax highlighting and code parsing |
| **roslyn.nvim** | Roslyn LSP for .NET with Razor/CSHTML support |

## Key bindings

`Space` is the leader key.

### General

| Key | Action |
|-----|--------|
| `<Esc>` | Clear search highlight |
| `<leader>w` | Toggle line wrap |
| `<leader>tg` | Toggle line numbers (git signs stay visible) |
| `<leader>bd` | Delete current buffer |

### Window Navigation

| Key | Action |
|-----|--------|
| `Ctrl+h` | Move to left window |
| `Ctrl+j` | Move to lower window |
| `Ctrl+k` | Move to upper window |
| `Ctrl+l` | Move to right window |

### LSP (Language Server)

| Key | Action |
|-----|--------|
| `gd` | Go to definition |
| `gD` | Go to declaration |
| `gr` | Find references |
| `gi` | Go to implementation |
| `K` | Show hover documentation |
| `<leader>rn` | Rename symbol |
| `<leader>ca` | Code actions |
| `<leader>f` | Format file |

### Telescope (Fuzzy Finder)

| Key | Action |
|-----|--------|
| `<leader>ff` | Find files |
| `<leader>fg` | Live grep (search in files) |
| `<leader>fb` | Find open buffers |
| `<leader>fh` | Search help tags |
| `<leader>fk` | Find keymaps |
| `<leader>fp` | Print full file path |
| `<leader>fr` | Recent files |

### Git (Gitsigns)

| Key | Action |
|-----|--------|
| `]h` | Next hunk |
| `[h` | Previous hunk |
| `<leader>hp` | Preview hunk |
| `<leader>hs` | Stage hunk |
| `<leader>hr` | Reset hunk |
| `<leader>hd` | Diff this |

### Flash

| Key | Action |
|-----|--------|
| `s` | Flash jump |
| `S` | Flash Treesitter |
| `r` | Remote Flash (operator-pending) |
| `R` | Treesitter search |
| `Ctrl+s` | Toggle Flash search (command-line) |

### Git Diff

| Key | Action |
|-----|--------|
| `<leader>gs` | Git status (fuzzy find changed files) |
| `<leader>diff` | Show unstaged changes + new files |
| `<leader>dic` | Show staged changes (git diff --cached) |

### Autocompletion

| Key | Action |
|-----|--------|
| `Tab` | Next completion item / expand snippet |
| `Shift+Tab` | Previous completion item |
| `Enter` | Accept completion |
| `Ctrl+Space` | Trigger completion manually |
| `Ctrl+e` | Abort completion |
| `Ctrl+b` | Scroll docs up |
| `Ctrl+f` | Scroll docs down |

### Smooth Scrolling

| Key | Action |
|-----|--------|
| `Ctrl+u` | Scroll up (half page) |
| `Ctrl+d` | Scroll down (half page) |
| `Ctrl+b` | Scroll up (full page) |
| `Ctrl+f` | Scroll down (full page) |
| `zt` | Scroll cursor to top |
| `zz` | Scroll cursor to center |
| `zb` | Scroll cursor to bottom |

### Visual Mode

| Key | Action |
|-----|--------|
| `<` | Indent left (stay in visual) |
| `>` | Indent right (stay in visual) |
| `J` | Move selected lines down |
| `K` | Move selected lines up |


## Tmux Configuration

```
tmux/
├── .tmux.conf
└── scripts/
    └── fuzzy_insert_path.sh  # Two-step fuzzy insert: dir picker, optional file picker (Tab)
```

### Features

- Prefix key: `Ctrl+a` (instead of default `Ctrl+b`)
- Vi mode enabled
- Mouse support
- 10000 line scrollback history
- Windows/panes start at index 1

### Plugins (via TPM)

| Plugin | Description |
|--------|-------------|
| **tpm** | Tmux Plugin Manager |
| **tmux-sensible** | Sensible defaults |
| **tmux-resurrect** | Save/restore sessions |
| **tmux-copycat** | Enhanced search |

### Key bindings

| Key | Action |
|-----|--------|
| `Ctrl+a` | Prefix (send with `Ctrl+a Ctrl+a`) |
| `Prefix r` | Reload config |
| `Prefix v` | Split vertically |
| `Prefix h` | Split horizontally |
| `Prefix T` | Move window to position 1 |
| `Alt+c` | Fuzzy insert path from dir history — Enter inserts dir, Tab opens file picker under selected dir (no prefix) |
| `Prefix j` | Scroll down (page) |
| `Prefix k` | Scroll up (page) |
| `Alt+z` | Zoom/unzoom current pane (no prefix) |
| `Alt+s` | Enter copy mode and search backward (no prefix) |

### Pane Navigation

| Key | Action |
|-----|--------|
| `Alt+Left` | Select pane left |
| `Alt+Right` | Select pane right |
| `Alt+Up` | Select pane up |
| `Alt+Down` | Select pane down |

### Window Navigation

| Key | Action |
|-----|--------|
| `Shift+Left` | Previous window |
| `Shift+Right` | Next window |
| `Alt+1`..`Alt+9` | Go to window 1-9 |
| `Alt+0` | Go to window 10 |


## Fish Configuration

```
fish/
├── config.fish          # Main config: PATH, abbreviations, aliases, env vars
├── conf.d/
│   ├── omf.fish         # Oh My Fish loader
│   └── rustup.fish      # Cargo/Rust env
└── functions/
    ├── fcd.fish         # Fuzzy cd from dir history (Ctrl+g)
    ├── fish_prompt.fish # Custom prompt with git/hg branch and dirty indicator
    └── __log_dir.fish   # Logs every visited dir to dir_history on PWD change
```

### Notable abbreviations / aliases

| Abbr/Alias | Expands to |
|------------|-----------|
| `gs` | `git status` |
| `ga` | `git add -A` |
| `ss` / `sp` | `git stash save -u` / `git stash pop` |
| `d` / `dc` / `dps` | docker / docker compose / docker ps |
| `p` / `pc` | podman / podman compose |
| `k` / `kl` | kubectl / kubectl logs |
| `dn` | dotnet |
| `v` | `nvim` |
| `cv` | `claude` |
| `l` | `ls -lA` |
| `Ctrl+g` | `fcd` — fuzzy jump to dir from history |


## Claude Code Configuration

```
claude-code/
└── commands/             # User-level slash commands (linked to ~/.claude/commands)
```

---
> Source: [gornostal/configs](https://github.com/gornostal/configs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
