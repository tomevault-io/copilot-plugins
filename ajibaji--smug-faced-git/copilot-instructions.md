## smug-faced-git

> This is a **dotfiles configuration repository** that manages terminal and development environment configurations across macOS and Linux using [Dotbot](https://github.com/anishathalye/dotbot).

# AGENTS.md

This is a **dotfiles configuration repository** that manages terminal and development environment configurations across macOS and Linux using [Dotbot](https://github.com/anishathalye/dotbot).

## Quick Start

```bash
# Clone and run full setup (interactive menu)
./go.sh

# Or run directly:
./dotbot-run.sh           # Full setup (symlinks + OS-specific configs)
./dotbot-run.sh symlink   # Symlinks only (no package installation)
```

## Repository Structure

```
.
├── go.sh / go.ps1           # Interactive setup menu (macOS/Linux / Windows)
├── dotbot-run.sh            # Main entry point - runs Dotbot with configs
├── base.conf.yaml           # Symlink definitions (cross-platform)
├── mac.conf.yaml            # macOS-specific: Homebrew + system defaults
├── linux-base.conf.yaml     # Linux-specific: shell commands
├── apt.conf.yaml            # Debian/Ubuntu apt packages
├── pre-run.sh               # Linux pre-install (apt repos, cargo, fnm, dotnet)
├── post-run.sh              # Post-install tasks
│
├── dotbot/                  # Git submodule - Dotbot core
├── dotbot-brew/             # Git submodule - Homebrew plugin
│
└── <app>/                   # Per-application config directories
    └── config files...
```

## Key Configuration Directories

| Directory | Purpose |
|-----------|---------|
| `nvim/` | Neovim config (Lua, Lazy.nvim plugin manager) |
| `zsh/` | Zsh shell config (`.zshrc`, `.zprofile`, `.zshenv`) |
| `bash/` | Bash shell config |
| `shell/` | Shared shell config (env vars, aliases, functions) |
| `aerospace/` | AeroSpace tiling window manager (macOS) |
| `kitty/` | Kitty terminal config |
| `ghostty/` | Ghostty terminal config |
| `lazygit/` | LazyGit config (with light/dark themes) |
| `git-config/` | Git global config and ignore patterns |
| `brew/` | Homebrew Brewfile (macOS packages) |
| `starship/` | Starship prompt config |
| `yazi/` | Yazi file manager config |
| `crush/` | Crush AI assistant config |

## How Dotbot Works

1. **`base.conf.yaml`** defines symlinks from `~/.config/<app>` → repository directories
2. **`mac.conf.yaml`** runs Homebrew via `Brewfile` and configures macOS defaults
3. **`linux-base.conf.yaml`** runs Linux-specific setup commands
4. **`apt.conf.yaml`** installs Debian/Ubuntu packages

Symlinks are created with `force: true` and `relink: true`, meaning they will overwrite existing files.

## Shell Configuration

The shell setup uses a layered approach:

```
zsh/.zshenv    → Loaded first (minimal, sets XDG paths)
zsh/.zprofile  → Login shell setup
zsh/.zshrc     → Interactive shell (sources .zshroot)
zsh/.zshroot   → Common interactive config
shell/env.sh   → Environment variables (PATH, tool configs)
shell/lib/aliases.sh → Shell aliases (bat, docker, git, k8s, vim)
```

### Key Environment Variables

- `EDITOR=nvim`
- `CURRENT_THEME` - Controls light/dark theme switching (used by lazygit, delta, nvim)
- `XDG_CONFIG_HOME=$HOME/.config`

### Notable Aliases

```bash
v='nvim'           # Neovim
lg='lazygit'       # LazyGit
kk='kubectl'       # Kubernetes
tt='toggle-theme'  # Theme switcher
gst='git status'
gco='git checkout'
```

On macOS, GNU coreutils are aliased to replace BSD versions (e.g., `ls='gls'`, `sed='gsed'`).

## Neovim Configuration

Located in `nvim/`, uses **Lazy.nvim** as the plugin manager.

```
nvim/
├── init.lua                  # Entry point
└── lua/
    ├── config/               # Core settings
    │   ├── options.lua       # Vim options
    │   ├── keymaps.lua       # Key bindings
    │   ├── autocommands.lua  # Auto commands
    │   └── constants.lua     # Shared constants
    ├── lazy/                 # Lazy.nvim setup
    │   └── lazy-plugins.lua  # Plugin specifications
    ├── plugins/              # Plugin configs by category
    │   ├── flow/             # Workflow (autopairs, copilot, git, etc.)
    │   ├── lsp/              # LSP, completion, treesitter, formatting
    │   └── visual/           # UI (theme, statusbar, notifications)
    └── utils/                # Utility functions
```

Theme is controlled by `CURRENT_THEME` env var (values: `DARK` or `LIGHT`).

## Git Configuration

- Uses **delta** for diffs with side-by-side view
- Theme-aware diff colors via `delta.features` env var
- Global gitignore at `git-config/.gitignore_global`

## Submodules

The repo includes these Git submodules:

- `dotbot/` - Core Dotbot framework
- `dotbot-brew/` - Homebrew plugin for Dotbot

To update submodules:
```bash
git submodule update --init --recursive
```

## Platform-Specific Notes

### macOS

- Uses **Homebrew** for package management (`brew/Brewfile`)
- **AeroSpace** tiling window manager with vim-like keybindings
- **Sketchybar** status bar
- Extensive `defaults write` commands in `mac.conf.yaml` for system preferences
- GNU coreutils installed and aliased to replace BSD versions

### Linux

- Uses **apt** for package management
- Installs from pre-run.sh: cargo/rust, fnm/node, dotnet
- Sets up Docker, Kubernetes, Azure CLI apt repositories
- Kitty terminal installed manually (not via apt)

### WSL

- Sets `KITTY_DISABLE_WAYLAND=1`
- Configures `DISPLAY` for X11 forwarding

## Adding New Configurations

1. Create a directory for the application: `mkdir <app>/`
2. Add config files to the directory
3. Add symlink entry to `base.conf.yaml`:
   ```yaml
   - link:
       ~/.config/<app>: <app>
   ```
4. Run `./dotbot-run.sh symlink` to create the symlink

## Common Tasks

### Add a new Homebrew package (macOS)

Edit `brew/Brewfile`:
```ruby
brew "package-name"
# or
cask "app-name"
```

### Add a new apt package (Linux)

Edit `apt.conf.yaml`:
```yaml
- apt:
    - package-name
```

### Modify shell aliases

Edit `shell/lib/aliases.sh`

### Add shell environment variables

Edit `shell/env.sh`

## Gotchas

1. **Submodules must be initialized** before running Dotbot - `dotbot-run.sh` handles this automatically
2. **Symlinks are forcefully overwritten** - existing files at target paths will be replaced
3. **macOS coreutils aliases** may break scripts expecting BSD behavior - check `aliases.sh`
4. **Theme switching** requires `CURRENT_THEME` env var to be set before launching apps
5. **Neovim plugins** are managed by Lazy.nvim, not system packages
6. **git stash** is automatically used during `dotbot-run.sh` if there are uncommitted changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ajibaji)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ajibaji)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
