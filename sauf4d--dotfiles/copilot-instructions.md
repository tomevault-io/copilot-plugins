## dotfiles

> A cross-platform, profile-based zsh configuration system. Ships on macOS and common Linux

# CLAUDE.md — Dotfiles Development Guide

## Project Overview

A cross-platform, profile-based zsh configuration system. Ships on macOS and common Linux
distros. Keeps shell startup under 200ms by lazy-loading heavy tools.

Three cumulative profiles:

| Profile   | Tools added |
|-----------|-------------|
| `minimal` | tmux (+ sheldon infrastructure) |
| `server`  | bat, eza, fd, fzf, ripgrep, tealdeer, zoxide, vfox |

---

## Shell Language Rules

| Path | Language | Reason |
|------|----------|--------|
| `bin/dotfiles` | **Bash** | Runs before zsh is configured; must work on minimal systems |
| `zsh/**/*.zsh` | **Zsh** | Shell config; may use zsh-specific builtins (`typeset`, `zstyle`, glob qualifiers) |

Do **not** mix the two. Never use `#!/usr/bin/env zsh` in `bin/dotfiles`.

---

## Architecture

```
bin/dotfiles          Bash CLI — symlinks, install, update, profile switch
  │
zshrc                 Entry point (< 40 lines) — sources core/ + libs + packages
  │
  ├── zsh/core/       Always-loaded modules (setopt, history, completion, aliases, theme)
  ├── zsh/lib/        Shared libraries — installer, platform detection
  └── zsh/packages/   One file per tool, grouped by profile tier
```

Full details: `docs/architecture.md`
Requirements: `docs/requirements.md`
How to add a package: `docs/guides/adding-a-package.md`

---

## Adding a New Package

1. Pick the right tier: `minimal` | `server`
2. Create **one file**: `zsh/packages/<tier>/<toolname>.zsh`
3. Do **not** modify `zshrc`, `installer.zsh`, or any other core file
4. Call `init_package_template "$PKG_NAME"` at the end

Minimal template:
```zsh
#!/usr/bin/env zsh

PKG_NAME="toolname"
PKG_DESC="One-line description"

pkg_init() {
    # runs every shell start — keep fast (< 5ms)
    alias t="toolname --flag"
}

init_package_template "$PKG_NAME"
```

---

## Naming Conventions

| Symbol | Convention | Example |
|--------|-----------|---------|
| Package variables | `PKG_NAME`, `PKG_DESC`, `PKG_CMD`, `PKG_CHECK_FUNC` | — |
| Hook functions | `pkg_init`, `pkg_install`, `pkg_pre_install`, `pkg_post_install`, `pkg_install_fallback` | — |
| Private helpers (check) | `_<tool>_is_installed` | `_nvm_is_installed` |
| Load flag (idempotency) | `_DOTFILES_<TOOL>_LOADED` | `_DOTFILES_VFOX_LOADED` |

---

## Idempotency Rules (Critical)

Shell startup must be **safe to run multiple times** (e.g. `source ~/.zshrc` after a tool
is already active). Any package with non-trivial `pkg_init` logic (sheldon, vfox) needs
a guard to prevent re-initialization:

```zsh
pkg_init() {
    [[ "${_DOTFILES_TOOL_LOADED:-}" == "1" ]] && return 0   # <-- required

    eval "$(tool activate zsh)"   # or other initialization

    export _DOTFILES_TOOL_LOADED="1"
}
```

> Without this guard: `source ~/.zshrc` re-runs the initialization, which can cause
> duplicate PATH entries, re-evaluated hooks, or degraded performance.

---

## Testing

```zsh
# Measure shell startup time (3-run average, discard first)
time zsh -i -c exit

# Verify all symlinks and package installs
dotfiles verify

# Test vfox works correctly
source ~/.zshrc
vfox --version   # should print version

# Re-source safety check (should produce no errors)
source ~/.zshrc
source ~/.zshrc
```

---

## Common Pitfalls

1. **Modifying core files** — Never add tool logic to `zshrc`, `installer.zsh`, or `zsh/core/*.zsh`.
   Each tool is self-contained in its own package file.

2. **Forgetting idempotency guards** — Packages with `eval` init (sheldon, vfox) need
   a load flag guard (see above). Skipping it causes hard-to-debug re-source breakage.

3. **Making `pkg_init` slow** — `pkg_init` runs synchronously at shell startup. Keep it
   under ~5ms. Use compiled tools (like vfox) that initialize quickly.

4. **Using `command -v` for shell-function tools** — Tools like `nvm` are not
   binaries. Set `PKG_CHECK_FUNC` to a custom function, or `command -v` will always fail.

5. **Adding a package that sorts before `00-sheldon.zsh`** — Files in `zsh/packages/minimal/`
   load alphabetically. Any new file starting with `a`–`n` would load before sheldon and
   break the plugin system. Prefix with a number (`00-`) if strict ordering is required.

6. **Bare `curl | bash` in `pkg_install_fallback`** — Always download to a temp file and
   verify a checksum before executing. See `docs/architecture.md` for the safe pattern.

---

## Key Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `DOTFILES_ROOT` | `~/.dotfiles` | Path to this repo |
| `DOTFILES_PROFILE` | `minimal` | Active profile (set via `dotfiles profile <name>`) |
| `DOTFILES_VERBOSE` | `false` | Set to `true` to trigger install flow + verbose logging |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sauf4d) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
