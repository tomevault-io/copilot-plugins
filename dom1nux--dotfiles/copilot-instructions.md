## dotfiles

> Reference for AI agents working in this dotfiles repository.

# Agents

Reference for AI agents working in this dotfiles repository.

## Repository purpose

Personal dotfiles managed with [chezmoi](https://www.chezmoi.io/), targeting Arch Linux on both bare metal and WSL. The goal is a single unified configuration across all machines — no hostname branching, no machine-specific conditionals.

## Chezmoi file naming conventions

| Prefix/suffix | Effect on target |
|---|---|
| `dot_` | Replaced with `.` (e.g. `dot_zshrc` → `~/.zshrc`) |
| `private_` | File is created with `0600` permissions |
| `executable_` | File is created with `+x` permissions |
| `.tmpl` suffix | Processed as a Go template before writing |
| `dot_config/` | Maps to `~/.config/` |

Plain files (no `.tmpl`) are copied as-is. Prefer plain files unless templating is actually needed.

## Key managed files

| Source | Target | Purpose |
|---|---|---|
| `dot_zshrc` | `~/.zshrc` | Zsh configuration — aliases, plugins, tool init |
| `dot_config/starship.toml` | `~/.config/starship.toml` | Starship prompt configuration |
| `dot_gitconfig` | `~/.gitconfig` | Git configuration |

## Commit convention

Format: `<type>(<scope>): <description>`

| Type | When to use |
|---|---|
| `feat` | New feature or configuration added |
| `fix` | Bug fix or broken config corrected |
| `refactor` | Restructuring without behavior change |
| `chore` | Maintenance, tooling, repo setup |
| `docs` | Documentation only |

Scopes follow the tool or file being changed: `zsh`, `starship`, `git`, `ghostty`, `hypr`, etc.

Examples from this repo:
```
feat(zsh): overhaul config with zinit, fzf, zoxide and modern cli tools
feat(starship): migrate from default config to OMP-inspired prompt
chore: init
```

## Shell config rules (`dot_zshrc`)

**Section order** (must be preserved):

1. Env vars and PATH exports — safe for non-interactive shells
2. Aliases — safe for non-interactive shells
3. Interactive guard: `[[ $- != *i* ]] && return`
4. History configuration
5. Keybindings
6. Zinit bootstrap + plugins
7. Completion setup (`compinit` + `zstyle`)
8. Tool init (`fzf`, `zoxide`, `starship`)
9. Functions
10. Fastfetch
11. `zcompile` performance block

**Rules:**
- Never place interactive-only code above the `[[ $- != *i* ]] && return` guard
- No hostname branching or machine-specific conditionals — single unified config
- JIT philosophy: only add aliases and config for tools actively in use
- `starship init zsh` must come after `fzf --zsh` and `zoxide init`
- `zsh-syntax-highlighting` must be the last sourced plugin (zinit handles this)
- The `zcompile` block must remain last

## Toolchain

| Role | Tool | Notes |
|---|---|---|
| Prompt | starship | Not OMP — OMP causes issues with some tools |
| Plugin manager | zinit | Turbo mode, auto-bootstraps on first launch |
| Fuzzy finder | fzf | Backend: `fd`, previewer: `bat` |
| Smart cd | zoxide | Initialized with `--cmd cd` (transparent replacement) |
| `ls` replacement | eza | With `--group-directories-first --icons=always` |
| `cat` replacement | bat | With `BAT_THEME=ansi` (respects terminal palette) |
| `find` replacement | fd | Used as fzf backend |
| `grep` replacement | rg (ripgrep) | Aliased with `--smart-case --color=always` |
| Editor | nvim | Aliased as `vi`; `vi()` function opens `nvim .` with no args |
| Git TUI | lazygit | Aliased as `lg` |
| Dotfiles manager | chezmoi | Aliased as `cz`; `cza` for apply |

## Starship prompt style

- **Style:** plain — no powerline glyphs, no background blocks
- **Left prompt:** `$os` icon + `◗▶` separator + `$directory` + `$character`
- **Right prompt:** `$git_branch$git_status` + language versions + `$cmd_duration`
- **Character module:** `●` — white on success, red on error
- Transient prompt is **not implemented** — known to conflict with `fzf-tab` and `zsh-autosuggestions` in zsh

## Colorscheme

Terminal colorscheme: **Catppuccin Mocha**.

- Prefer ANSI named colors (`red`, `yellow`, `cyan`, `blue`, `magenta`, `purple`, `bright-cyan`, etc.) — these are terminal-adaptive and will render as Mocha's palette automatically
- Use hex colors only when named ANSI colors lack sufficient contrast or don't map to the intended Mocha color (e.g. Node.js green `#417E38`, Rust orange `#CE422B`, Kubernetes blue `#326CE5`)
- The `[os]` module in starship supports only a single `style` — per-distro coloring via `[custom]` modules was attempted and reverted due to reliability issues; use a single named color instead

## What to avoid

- **Transient prompt in zsh** — conflicts with `fzf-tab` and `zsh-autosuggestions`, causes visual glitches
- **Hostname/machine branching** — the old config used `$isMizuki`/`$isMizuho` conditionals; this pattern is deprecated
- **OMP (oh-my-posh)** — replaced by starship due to compatibility issues with VSCode Copilot and other tools
- **Hardcoded hex colors** in starship where an ANSI named color is semantically equivalent
- **Pre-populating aliases** for tools not yet actively used — add aliases JIT when the tool becomes part of the workflow
- **Sourcing files at runtime** — inline config directly in `dot_zshrc` instead of splitting into separate sourced files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dom1nux)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dom1nux)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
