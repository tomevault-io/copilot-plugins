## tmux-simpre

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal tmux configuration distributed as two artifacts:

- `.tmux.conf` — the config users symlink to `~/.tmux.conf`
- `install.sh` — symlink installer

There is no build, no test suite, and no package. "Shipping" means committing edits to `.tmux.conf`; users pull and re-run `install.sh` (or just reload tmux, since the install is symlink-based).

## Install / reload workflow

```sh
bash install.sh                # symlinks .tmux.conf into $HOME (idempotent; prompts before overwriting)
tmux source-file ~/.tmux.conf  # or press prefix + r inside tmux (prefix is C-b)
```

Because `install.sh` uses a symlink, edits to `.tmux.conf` in this repo take effect on the next reload — re-running `install.sh` is only needed after a fresh clone or if the symlink is missing. For verification, `readlink ~/.tmux.conf` should point back into this repo.

To test changes without disturbing the user's running tmux, set `TMUX_TMPDIR=/tmp/tmux-test` and start a fresh `tmux -f ./.tmux.conf` server.

## The `prefix + g` lazygit popup

This is the only non-trivial binding in the config. `bind-key 'g'` runs an `if-shell` guard and opens lazygit in a popup only if the guard passes:

```tmux
bind-key 'g' if-shell \
  'command -v lazygit > /dev/null 2>&1 && git -C "#{pane_current_path}" rev-parse --is-inside-work-tree > /dev/null 2>&1' \
  'display-popup -E -w 80% -h 80% -S "fg=#b7bdf8" -s "bg=#2a2a2a" -d "#{pane_current_path}" "lazygit"' \
  ''
```

The guard requires **both** conditions: lazygit is installed, **and** the pane's current path is inside a git work tree. `git rev-parse --is-inside-work-tree` is used (not a literal `[ -d .git ]` test) so it also passes in **subdirectories** and in **git worktrees**, where `.git` is a file rather than a directory. When the guard fails, control falls through to the empty `''` branch — a deliberate silent no-op.

The guard line is evaluated by `/bin/sh` via tmux's `run-shell`, so keep it POSIX (`[ ... ]` tests, no bashisms).

## The `prefix + f` yazi toggle

`bind-key 'f'` toggles a [yazi](https://github.com/sxyazi/yazi) file-manager split: if the current pane is already running yazi it kills the pane; otherwise, if yazi is installed, it opens a horizontal split running yazi in the pane's current path. Like `prefix + g`, it no-ops silently when yazi isn't installed.

## Style conventions in this repo

- Catppuccin Macchiato palette via `#{@thm_*}` variables — reuse these instead of hardcoding hex colors when adding status-bar segments.
- Vim-style key bindings (`h/j/k/l` for pane nav, `v`/`s` for splits) are the user's mental model. New bindings should follow that idiom rather than the tmux defaults.
- Confirm-before for destructive keys (`x` window kill, `Q` session kill). New destructive bindings should do the same.

## Things that look like bugs but aren't

- `prefix + g` does nothing when lazygit isn't installed or the pane isn't inside a git repo — that's the empty `''` branch of `if-shell` and is intentional.
- `automatic-rename-format` shows the basename of the pane's cwd, which can make windows look identically named when you have multiple worktrees of the same repo open. That's a known trade-off, not a bug.

---
> Source: [Narong-Kanthanu/tmux-simpre](https://github.com/Narong-Kanthanu/tmux-simpre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
