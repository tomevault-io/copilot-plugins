## dotfiles

> Guidance for OpenCode sessions working in this repo.

# AGENTS.md

Guidance for OpenCode sessions working in this repo.

## Overview

Personal dotfiles — Emacs 30.2 config with Evil (vim) keybindings, plus a VSCode settings backup. Targets Python data science, Org mode, and ibuffer-based buffer management.

## Source of truth

The repo is the source of truth. Edit files here, not under `~/.emacs.d/`.

**Two deploy modes:**

- **A (primary)** — `~/.emacs.d/init.el` contains `(load "~/repos/dotfiles/emacs/init.el")`. Everything resolves from the repo path. No copy step.
- **B** — `bash emacs/move_files.sh` copies to `~/.emacs.d/` with a dated backup under `~/.emacs.d.bak/`.

## Emacs architecture

`emacs/init.el` resolves its own directory, loads `early-init.el` explicitly, adds `lisp/` to `load-path`, then `require`s modules in this dependency order:

| File | Purpose |
|---|---|
| `init-core.el` | MELPA, `use-package`, general settings (backups, auto-save, parens, electric modes, line numbers, hooks) |
| `init-ui.el` | Doom modeline |
| `init-tools.el` | Magit, Projectile, TRAMP, anzu, markdown-mode, aggressive-indent, indent-bars, ibuffer/ibuffer-project |
| `init-completion.el` | Corfu/Cape (in-buffer), Vertico/Orderless/Marginalia/Consult/Embark (minibuffer) |
| `init-evil.el` | Evil, evil-collection, evil-surround, evil-anzu, undo-fu/undo-fu-session/vundo |
| `init-python.el` | Tree-sitter grammars, `python-ts-mode` remap, `# %%` cell nav, venv detection, REPL |
| `init-org.el` | Org capture, babel, org-tempo, org-download, clocking |
| `init-daily-log.el` | Activity capture into `~/org/daily-log.org` + color-coded calendar/table views |

Disable any module by commenting out its `require` in `init.el`.

## Key conventions

- **No `custom-file`**: all settings are explicit `setq` inside `use-package` blocks. Nothing writes to `custom-set-variables`.
- **Python mode**: remapped via `major-mode-remap-alist` (`python-mode` → `python-ts-mode`), not by patching `auto-mode-alist`.
- **Evil keybindings**: use `evil-define-key` inside nested `with-eval-after-load 'evil ... with-eval-after-load '<package>` so deferred loading doesn't break them.
- **Cross-platform**: Python/venv paths use `system-type` checks and `executable-find` fallbacks.
- **`early-init.el`**: loaded explicitly by `init.el` (not auto-discovered). An `emacs-startup-hook` fallback switches to `*scratch*` after init.

## Known issues

- **`avy` not declared as dependency**: `init-evil.el` binds `SPC` to `avy-goto-char-2`, but there's no `(use-package avy ...)` anywhere. Will error on a clean install if `avy` wasn't pulled in transitively.
- **`<leader>` keybindings broken**: `init-python.el` binds `<leader>l`/`<leader>b`/`<leader>a` via `evil-define-key`, but these require `evil-leader` (not installed). Bindings silently do nothing.

## Tmux config

`tmux/tmux.conf` — deployed to `~/.config/tmux/tmux.conf` via `bash tmux/deploy.sh` (backs up existing first).

## Reference

- `emacs/README.md` — full architecture rationale, load order details, daily-log docs, and more known issues.
- `vscode/settings.json` — backed up VSCode config (not deployed by `move_files.sh`).
- `emacs/test/` and `emacs/notes/` — not loaded by Emacs.
- `.gitignore` ignores `*.elc`, `*~`, `#*#`, `.DS_Store`, and `eln-cache/`.

---
> Source: [huajie06/dotfiles](https://github.com/huajie06/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
