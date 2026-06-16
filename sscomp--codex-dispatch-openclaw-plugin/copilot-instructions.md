## codex-dispatch-openclaw-plugin

> Expose a `/codex` command in OpenClaw so users can dispatch structured

# Codex Dispatch

## Purpose

Expose a `/codex` command in OpenClaw so users can dispatch structured
coding tasks to a local Codex CLI instance.

## Includes

- OpenClaw extension for `/codex` and `/codex-projects`
- Background runner that executes `codex exec`
- Installer that copies files into an OpenClaw home and updates `openclaw.json`
- Project registry template for `/project` aliases

## Main install command

```bash
/Users/sscomp/codex-dispatch-openclaw-plugin/scripts/install-openclaw.sh /Users/sscomp/.openclaw
```

---
> Source: [sscomp/codex-dispatch-openclaw-plugin](https://github.com/sscomp/codex-dispatch-openclaw-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
