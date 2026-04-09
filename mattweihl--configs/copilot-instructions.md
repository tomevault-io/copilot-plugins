## configs

> Personal dotfiles for macOS/Linux. Cloned to `~/configs` (lowercase).

# Configs

Personal dotfiles for macOS/Linux. Cloned to `~/configs` (lowercase).

## Structure

```
ghostty/          Ghostty terminal config + custom icons
glow/             Glow markdown viewer config
iterm2/           iTerm2 profile, keymaps, color themes (legacy/backup)
lazygit/          Lazygit config + helper scripts
nvim/             Neovim config (Lua, lazy.nvim)
tmux/             tmux config
zsh/              zshrc
```

## How configs are deployed

- `nvim/` → symlinked to `~/.config/nvim/`
- `lazygit/` → symlinked to platform-specific lazygit config path
- `tmux/tmux.conf` → sourced from `~/.tmux.conf` wrapper file (not symlinked)
- `zsh/zshrc` → sourced from `~/.zshrc` wrapper file (not symlinked)
- `glow/` → symlinked to `~/Library/Preferences/glow/` (macOS) or `~/.config/glow/` (Linux)

## Neovim

- `init.lua` loads `core/` (options, keymaps, autocmds), then `lazy.setup("plugins")` auto-discovers `lua/plugins/*.lua`
- 2 spaces default indent, 4 for Python (autocmd)

## Conventions

- All config files should work on both macOS and Linux where possible
- Shell scripts in `lazygit/scripts/` use `~/configs/` as the base path
- No completion/autocomplete plugin in nvim (intentional)
- No format-on-save in nvim (intentional — format explicitly with `<leader>F`)
- Binary/generated files (`.icns`, `lazy-lock.json`, iTerm themes) are committed as-is

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattweihl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mattweihl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
