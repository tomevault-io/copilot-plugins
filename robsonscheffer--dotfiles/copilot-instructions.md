## dotfiles

> Personal dotfiles managed by [Chezmoi](https://www.chezmoi.io/). Targets macOS (darwin).

# AGENTS.md

Personal dotfiles managed by [Chezmoi](https://www.chezmoi.io/). Targets macOS (darwin).

## The Stack

| Layer | Component | Config source |
|-------|-----------|---------------|
| Terminal | Ghostty | `home/dot_config/ghostty/config` |
| Shell | zsh + Oh-My-Zsh | `home/dot_zshrc` |
| Prompt | oh-my-posh | `home/dot_config/oh-my-posh/theme.omp.json` |
| Editor | Neovim | `home/dot_config/nvim/` |
| Tools | mise | `home/dot_config/mise/config.toml` |
| Git | git | `home/dot_config/git/` |
| AI agents | Claude Code | `home/dot_claude/` (skills, hooks, settings) |
| Dashboard | gh-dash | `home/dot_config/gh-dash/config.yml.tpl` |

## Repo Map

```
home/                  # All managed dotfiles (Chezmoi source)
  dot_config/          #   -> ~/.config/
  dot_claude/          #   -> ~/.claude/
    hooks/             #     Pre/post tool hooks
    skills/            #     Slash-command skills
    settings.json      #     Claude Code settings
install.sh             # Bootstrap script
.githooks/             # Git pre-commit hooks for this repo
```

## The One Rule

**Edit in `home/`, never in `~` or `~/.config/`.** Chezmoi owns the home directory — destination edits get overwritten on next apply.

| To change... | Edit this |
|--------------|-----------|
| `~/.zshrc` | `home/dot_zshrc` |
| `~/.config/ghostty/config` | `home/dot_config/ghostty/config` |
| `~/.claude/settings.json` | `home/dot_claude/settings.json` |

## Change Workflow

1. Edit source files in `home/`
2. `chezmoi diff` — review changes
3. `chezmoi apply` — deploy to home directory

---
> Source: [robsonscheffer/dotfiles](https://github.com/robsonscheffer/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
