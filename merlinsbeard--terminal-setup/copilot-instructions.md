## terminal-setup

> Guidelines for AI coding agents working in this macOS dotfiles repository.

# AGENTS.md

Guidelines for AI coding agents working in this macOS dotfiles repository.

## Repository Overview

This is a **configuration-only** dotfiles repository for macOS. There is no build system, test suite, or compiled code. `./install.sh` installs required apps, `./deploy.sh` copies configs to their home directory locations.

## Commands

### Install Apps

```bash
./install.sh                     # Install all required apps and tools
./install.sh --help              # Show what gets installed
```

Installs: Homebrew, CLI tools (tmux, neovim, fzf, fd, ripgrep, etc.), Oh My Zsh, TPM, NVM, fonts.
Skips anything already installed.

### Deploy Configs (Go CLI - primary)

```bash
go run ./cmd/deploy/             # Interactive deploy with TUI
./deploy                         # Run pre-built binary
go build -o deploy ./cmd/deploy/ # Build the binary
```

Interactive Bubble Tea TUI with checkbox selection for targets. Supports:
- `space` to toggle targets, `a` to toggle all, `enter` to deploy
- Backup toggle (`b` on confirm screen)
- Catppuccin Mocha colored output

### Deploy Configs (Shell - fallback)

```bash
./deploy.sh                      # Deploy all configs (with backup)
./deploy.sh zsh nvim tmux        # Deploy specific targets
./deploy.sh --no-backup          # Deploy without backing up existing files
./deploy.sh --list               # List available targets
./deploy.sh --help               # Show usage
```

Targets: `zsh`, `nvim`, `tmux`, `aerospace`, `ghostty`, `git`, `vscode`, `zshenv`

The script backs up existing configs to `~/.dotfiles-backup/<timestamp>/` before overwriting.
`zshenv` is only copied if `~/.zshenv` doesn't already exist (to protect secrets).
Warns if a target app is not installed (but still copies the config).

### Verification Commands

```bash
# Validate shell script syntax
bash -n install.sh deploy.sh
zsh -n zshrc

# Reload configs after deployment
# AeroSpace: alt-shift-; then esc
# tmux: prefix + I (Ctrl-a + I) to install plugins, or restart session
# zsh: source ~/.zshrc or open new terminal
```

### CI - GitHub Actions

A security workflow runs on push to `main` and on pull requests (`.github/workflows/security.yml`):

- **Secret scanning** - gitleaks detects leaked tokens, API keys, passwords
- **ShellCheck** - lints `install.sh`, `deploy.sh`, `zshrc` for unsafe shell patterns
- **Dangerous patterns** - flags `rm -rf /`, `chmod 777`, `curl | sudo`, etc.
- **Syntax validation** - runs `bash -n` and `zsh -n` on scripts

## File Structure

```
.
├── install.sh            # Install required apps and tools
├── deploy.sh             # Deploy configs to home directory
├── .github/workflows/    # CI security checks (gitleaks, shellcheck)
├── aerospace.toml        # AeroSpace tiling window manager
├── config.ghostty        # Ghostty terminal emulator
├── gitconfig             # Git configuration
├── tmux.conf             # tmux terminal multiplexer
├── zshrc                 # Zsh shell configuration
├── zshenv.example        # Environment variables template (secrets)
├── nvim/                 # Neovim config (LazyVim framework)
│   ├── init.lua          # Entry point
│   ├── stylua.toml       # Lua formatter config
│   └── lua/
│       ├── config/       # LazyVim overrides
│       └── plugins/      # Plugin specifications
├── zsh/                  # Cached completions directory
├── fonts/                # Font files
└── vscode-settings-user.json
```

## Code Style Guidelines

### General Conventions

- **Theme**: Catppuccin (Mocha/Macchiato) across all tools. Do not introduce other themes.
- **No emojis in config files**: Only use emojis where already present (tmux status bar).
- **Secrets handling**: Never commit real tokens. Use `zshenv.example` as template only.

### TOML Files (aerospace.toml)

- Use single quotes for strings: `'tiles'`, `'main'`
- Comments start with `#`
- TOML formatter may strip section indentation to flat style - this is acceptable
- Array tables use `[[table.name]]` syntax
- Inline arrays for simple lists: `run = ['command1', 'command2']`

### Shell Scripts (install.sh, deploy.sh)

- `set -euo pipefail` at the top
- Catppuccin Mocha truecolor output with `$NO_COLOR` and tty detection fallback
- Use `printf` with `%-Ns` for column-aligned output
- Section headers with `# ===...===` blocks (same as zshrc)
- Use `$SCRIPT_DIR` for paths relative to the script, not `$PWD`
- Check `command -v` for app detection, `brew list` for package detection

### Shell Files (zshrc)

- Use section headers with `# ===...===` blocks
- Group related settings together
- Lazy-load heavy tools (NVM, Conda) - never add eager loading
- Function naming: `_private_func()` for internal, `public_func()` for user-facing
- Alias format: `alias short='longer command'`
- Use `$HOME` or `~` consistently (prefer `$HOME` in scripts)

```bash
# Good - lazy loading pattern
nvm() { _nvm_load; nvm "$@"; }

# Good - section organization
# =============================================================================
# Section Name
# =============================================================================
```

### tmux Configuration

- Comment sections with `# ==========================`
- Prefix is `Ctrl-a` (C-a), not default `Ctrl-b`
- Use lowercase for set commands: `set -g`, `set-option -g`
- Plugin format: `set -g @plugin 'author/plugin-name'`

### Git Configuration

- Sections in brackets: `[section]`
- Use tabs for indentation after section headers
- Add comments explaining non-obvious settings

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Aliases | Short, lowercase | `gm`, `nd`, `vim` |
| Functions | snake_case | `load_nvmrc`, `_nvm_load` |
| Environment vars | UPPER_SNAKE_CASE | `NVM_DIR`, `FZF_DEFAULT_COMMAND` |
| tmux options | kebab-case | `status-left-length`, `pane-border-status` |

## Key Architecture Decisions

1. **Install script**: `./install.sh` installs apps; `./deploy.sh` copies configs with backup; not symlinked
2. **Lazy loading**: NVM and Conda load on-demand to keep shell startup fast (~150ms target)
3. **Cached completions**: kubectl, gh, uv completions cached in `~/.zsh/` directory
4. **LazyVim framework**: Neovim uses LazyVim - extend via `lua/plugins/`, don't modify core
5. **Alt as modifier**: AeroSpace uses `alt` as primary modifier, `alt-shift` for secondary

## Common Pitfalls

- **Don't add eager-loading**: Heavy tools must be lazy-loaded in zshrc
- **Don't change tmux prefix**: Must remain `Ctrl-a` for muscle memory
- **Don't commit .zshenv**: Only `zshenv.example` is tracked; real secrets stay local
- **Don't modify LazyVim core**: Add overrides in `lua/plugins/`, import extras properly
- **Check deployment**: After editing, run `./deploy.sh <target>` to apply changes
- **Don't add dangerous patterns**: CI flags `rm -rf /`, `chmod 777`, `curl | sudo`, etc.

## File-Specific Notes

### aerospace.toml
- `alt-shift-;` enters service mode, `esc` exits and reloads
- App launchers: `alt-g` (Ghostty), `alt-b` (Brave), etc.
- Floating apps defined in `[[on-window-detected]]` sections

### tmux.conf
- TPM (plugin manager) installed via `./install.sh`
- After changes: `Ctrl-a + I` to install/update plugins

### zshrc
- Profile startup: uncomment `zmodload zsh/zprof` at top and `zprof` at bottom
- Oh-My-Zsh plugins minimized for speed - add sparingly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/merlinsbeard)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/merlinsbeard)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
