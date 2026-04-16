## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Personal dotfiles managed by [chezmoi](https://chezmoi.io). The source directory (`~/.local/share/chezmoi`) maps to `~` via chezmoi's naming conventions — `dot_` becomes `.`, `private_` sets restrictive permissions, and `.tmpl` files are rendered as Go templates before being written.

## Common chezmoi commands

```bash
chezmoi diff                  # preview what would change in ~
chezmoi apply                 # apply source → home
chezmoi add ~/.config/foo     # bring a new file under management
chezmoi edit ~/.config/foo    # edit the source file for a target
chezmoi doctor                # diagnose environment issues
```

## File naming conventions

| Source name | Target name | Meaning |
|-------------|-------------|---------|
| `dot_foo` | `.foo` | dotfile |
| `private_dot_config/` | `~/.config/` | chmod 600 |
| `foo.tmpl` | `foo` (rendered) | Go template, evaluated at apply time |
| `run_once_*.sh` | (not copied) | runs once on first `chezmoi apply` |

## Templates and data

`.chezmoi.toml.tmpl` prompts for one variable at `chezmoi init` time:

- **`has_sudo`** (bool) — stored in `~/.config/chezmoi/chezmoi.toml` after first init; controls whether the install script runs `apt install` or just prints a manual package list.

Template variables available in `.tmpl` files:
- `.chezmoi.os` — `"darwin"` or `"linux"`
- `.has_sudo` — from the prompt above

Files that use templates: `config.fish.tmpl`, `run_once_install-packages.sh.tmpl`.

Shell fallback files:
- `dot_bashrc` -> `~/.bashrc`
- `dot_zshrc` -> `~/.zshrc`
- `dot_shell_common` -> `~/.shell_common`, sourced by both bash and zsh for shared PATH, aliases, zoxide, oh-my-posh, and Rust setup

## Install script behaviour

`run_once_install-packages.sh.tmpl` is rendered and run once per machine:
- **macOS** — installs Homebrew if missing, then installs all packages via `brew`
- **Linux, `has_sudo = true`** — installs via `apt`
- **Linux, `has_sudo = false`** — skips installation, prints the package list

## macOS-specific config

Anything touching `/opt/homebrew` is wrapped in `{{ if eq .chezmoi.os "darwin" }}` in the template files. On Linux, Homebrew paths are omitted. Tmux does not set `default-shell`; it uses tmux's default behavior based on the shell environment when the server starts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cmll21) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
