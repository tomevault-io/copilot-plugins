## zellij

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Personal Zellij terminal multiplexer configuration. The repo contains a single `config.kdl` file with custom keybindings, plugin settings, and preferences. No build, lint, or test commands — this is a dotfiles-style config repo.

## Config Structure (config.kdl)

- **Keybinds**: `clear-defaults=true` — all default bindings are removed; everything is explicitly defined. Uses vim-style hjkl navigation throughout all modes.
- **Mode switching**: `Ctrl` + letter enters modes (p=pane, t=tab, n=resize, h=move, s=scroll, o=session, b=tmux, g=locked). `Enter` or `Esc` returns to normal.
- **Autolock plugin**: `zellij-autolock` (loaded from GitHub releases) auto-locks Zellij when nvim/vim opens and watches fzf/zoxide/atuin. Normal mode intercepts `Enter` to message the autolock plugin.
- **Theme**: `pencil-light`
- **Startup tips disabled**

## KDL Format Notes

When editing `config.kdl`, preserve KDL syntax: `node key=value { children }`. Bindings use `bind "key" { Action; }` — actions are semicolon-terminated inside braces. `shared_except` and `shared_among` blocks define bindings across multiple modes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vyorkin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vyorkin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
