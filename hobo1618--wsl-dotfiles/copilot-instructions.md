## wsl-dotfiles

> - Root contains macOS dotfiles; each directory matches its target tool for straightforward symlinks.

# Repository Guidelines

## Project Structure & Module Organization
- Root contains macOS dotfiles; each directory matches its target tool for straightforward symlinks.
- `fish/` bootstraps the shell (aliases, llm helpers); `.hammerspoon/` automates mac hotkeys, while `tmux/`, `alacritty/`, `.amethyst.yml`, `karabiner/`, and `starship.toml` tune terminal, tiling, and key remaps.
- Keep telemetry and CLI tweaks in `gh/config.yml` and `nextjs-nodejs/config.json`; avoid storing personal IDs beyond the placeholders already tracked.

## Build, Test, and Development Commands
- `fish --private --init-command 'source $PWD/fish/config.fish'` sanity-checks syntax without touching the current shell.
- `tmux source-file $PWD/tmux/tmux.conf` refreshes a running session; follow with `prefix + r` to confirm key bindings.
- `hs -c 'hs.reload()'` reloads Hammerspoon; look for the "Hammerspoon reloaded" toast before relying on new bindings.
- `karabiner_cli --reloadxml` and `alacritty msg config reload` apply keyboard and terminal edits; reopen the apps if commands fail silently.

## Coding Style & Naming Conventions
- Keep functions and aliases snake-case and descriptive (`llm-list`, `notes`), grouping related entries together in `fish/config.fish`.
- Use four-space indentation in Fish scripts, preserve tabs inside `.hammerspoon/init.lua`, and maintain existing alignment in TOML, YAML, and JSON files.
- Document non-obvious workarounds with short comments that match the current tone (e.g., BANK_DIR guardrails).
- Check in only ASCII and prefer alphabetised tables when adjusting larger colour or plugin lists.

## Testing Guidelines
- Launch a fresh Fish shell (`fish --login`) and run `type llm-list` to ensure new helpers resolve.
- After tmux edits, split panes and cycle with `hjkl`; confirm TPM still autoloads after `prefix + I`.
- Trigger leader shortcuts (`⌘; s b/c/p`) to verify screenshot flows and submodal timers; capture a test shot to confirm BANK_DIR usage.
- Use Karabiner EventViewer to confirm remapped keys fire, and reopen Alacritty to verify palette or font adjustments.

## Commit & Pull Request Guidelines
- Follow the short, imperative commit subjects already in history (`add hammerspoon`, `update env vars for macos`).
- Limit each pull request to one tool or behavioural change; include a brief rationale, testing notes, and screenshots for visual tweaks.
- Link any Askerra tracking issue and tag another macOS user for review when hardware-specific behaviour is touched.

## Security & Configuration Tips
- Keep secrets and machine-specific paths out of git; rely on environment variables and documented placeholders instead.
- Probe for optional dependencies (`bun`, `sqlite3`, `fzf`) before invoking them, echoing clear guidance when falling back.

---
> Source: [hobo1618/wsl-dotfiles](https://github.com/hobo1618/wsl-dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
