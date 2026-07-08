## dotfiles

> **RULE #1 — edit configs HERE, never under `$HOME`.** Nearly every config on this machine (`~/.config/tmux`, `~/.config/nvim`, `~/.zshrc`, …) is a symlink into this repo. When asked to change any tool's configuration, do NOT go looking in the home directory — find and edit the corresponding file in this repo (e.g. "edit tmux config" → `.config/tmux/` here). The full symlink list is in "What this repo is" below.

# AGENTS.md

**RULE #1 — edit configs HERE, never under `$HOME`.** Nearly every config on this machine (`~/.config/tmux`, `~/.config/nvim`, `~/.zshrc`, …) is a symlink into this repo. When asked to change any tool's configuration, do NOT go looking in the home directory — find and edit the corresponding file in this repo (e.g. "edit tmux config" → `.config/tmux/` here). The full symlink list is in "What this repo is" below.

Dotfiles repo synced across macOS systems. No build, no test suite — changes are verified by sourcing the relevant config or restarting the affected app.

## What this repo is

Files live here in their final on-disk layout (e.g. `.config/nvim/`, `.zshrc`). `install.sh` symlinks each entry of its `dotfiles=()` array from `$HOME` back into this repo. So editing a file under this repo is the same as editing the live config on the machine running `install.sh`.

This matters for any request to change tooling config: the live files in `$HOME` (`~/.config/nvim`, `~/.config/opencode`, `~/.zshrc`, …) are symlinks pointing here, so the source of truth is this repo — edit the file here, not a copy under `$HOME`. Directory entries are linked as **whole directories**, so everything inside them is live too.

Managed symlinks (`$HOME/<path> → repo/<path>`, from `install.sh`):

- Files: `.gitconfig`, `.gitignore`, `.ideavimrc`, `.profile.common`, `.zprofile`, `.zshrc`, `.stylua.toml`, `.editorconfig`, `.npmrc`, `.Shadowsocks-NG/user-rule.txt`
- Directories: `.config/ghostty`, `.config/nvim`, `.config/bat`, `.config/linearmouse`, `.config/lsd`, `.config/zim`, `.config/starship`, `.config/opencode`, `.config/tmux`
- Special case: `~/.profile` is **not** in the array — `install.sh` links it separately to `.profile.mac`.

## install.sh — read before changing anything top-level

- The `dotfiles=()` array at the top of `install.sh` is the **only** registry of managed paths. New dotfile? Add it there or it will not be symlinked on a fresh machine.
- `./install.sh --dry-run` prints what it would do without touching the filesystem. Use this to verify additions.
- A real run is destructive: existing non-symlink targets in `$HOME` are moved to `~/dotfiles-backup/`. Symlinks at the target are removed and recreated.
- iTerm2 prefs are **not** in the `dotfiles=()` array. They are wired via `defaults write` pointing at `~/dotfiles/iterm` (see `setup_iterm()`).

## Machine-specific / secret config (not in repo, do not commit)

- `~/.profile-local` and `~/.profile-specific` are sourced by `.profile` if present. The repo ships `.profile-specific.work` as a template — it is **not** symlinked; copy it to `~/.profile-specific` manually on a work machine.
- `~/.gitconfig_local` holds the `[user]` block (name/email) and is included from `.gitconfig`. Not in repo.
- `CLAUDE.local.md`, `out/`, `_local/` are gitignored.

## Nested instructions — respect them

- `.config/nvim/AGENTS.md` — full conventions for the Neovim config (lazy.nvim spec format, SSH-only plugin URLs, stylua, no test runner — verify by restarting nvim). `.config/nvim/CLAUDE.md` sits alongside it.

When editing files inside `.config/nvim/`, defer to that nested file for conventions.

Note: `.config/opencode/AGENTS.md` is **not** an instruction file for this repo — it is content shipped as the user's global OpenCode instructions on the target machine. Treat it as a regular tracked file; only read it if the task is actually about its contents.

## Style

- `.editorconfig` is authoritative: 4-space indent, LF, final newline. 2 spaces for `*.{yaml,yml,json,jsonc,c,scala}`.
- Lua is formatted with `stylua` (`.stylua.toml`: 4 spaces, `collapse_simple_statement = "FunctionOnly"`).
- No repo-wide formatter/linter command — formatters are invoked from inside the relevant editor (e.g. conform.nvim for the nvim config).

## Verifying changes

- Shell config (`.zshrc`, `.profile`, `.zprofile`): open a new shell, or `exec zsh`. Note `.zshrc` short-circuits inside JetBrains terminals (`INTELLIJ_ENVIRONMENT_READER`) and `cd $HOME`s on launch outside tmux/JetBrains.
- Neovim: restart or `:source %`; see `.config/nvim/AGENTS.md`.
- Anything driven by `defaults write` (iTerm): requires app restart.

## Things an agent will get wrong without this file

- Adding a new dotfile and forgetting `install.sh`'s array.
- Trying to commit `~/.gitconfig_local` content into `.gitconfig`.
- Editing `.profile-specific.work` expecting it to be live (it isn't — `.profile` reads `~/.profile-specific`).
- Looking for a test/lint/CI command. There is none.
- Assuming files in `$HOME` are the source of truth — the repo is, via symlinks created by `install.sh`.

---
> Source: [ruzveld83/dotfiles](https://github.com/ruzveld83/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
