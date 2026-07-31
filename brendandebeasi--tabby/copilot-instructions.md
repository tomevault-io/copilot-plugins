## tabby

> - [Dev Reload Workflow (Opt-in)](#dev-reload-workflow-opt-in)

# Claude Dev Notes

## Table of Contents

- [Dev Reload Workflow (Opt-in)](#dev-reload-workflow-opt-in)

## Dev Reload Workflow (Opt-in)

- Enable: `tmux set-option -g @tabby_dev_reload_enabled 1`
- Source config: `tmux source ~/.tmux.conf`
- Trigger reload: `prefix + R`
- Script: `bin/tabby-dev reload` (rebuilds binaries, restarts sidebar only if enabled)

### Behavior

- No-op if dev reload is disabled.
- Rebuilds via `scripts/install.sh`.
- Preserves sidebar state (does not enable if disabled).
- On client attach, auto-reloads when enabled.

---
> Source: [brendandebeasi/tabby](https://github.com/brendandebeasi/tabby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
