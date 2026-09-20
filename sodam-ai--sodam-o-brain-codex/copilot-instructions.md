## sodam-o-brain-codex

> O-Brain is a local-first personal memory plugin for Codex. It captures confirmed user decisions at `Stop`, recalls related memories at `SessionStart`, exposes seven MCP tools, and serves a local dashboard on `127.0.0.1`.

# O-Brain for Codex — Repository Guide

## Product

O-Brain is a local-first personal memory plugin for Codex. It captures confirmed user decisions at `Stop`, recalls related memories at `SessionStart`, exposes seven MCP tools, and serves a local dashboard on `127.0.0.1`.

## Source of truth

Read in this order:

1. `README.md` — current installation, use, security, and legal guidance
2. `.PRD/13_CODEX_PORT.md` — Codex port decisions and verification gates
3. `.PRD/README.md` and numbered PRDs — product and historical design details
4. Current code and tests — executable behavior

Claude-specific paths in older PRDs are historical. Do not restore the removed `plugin/.claude-plugin` layout in this Codex port.

## Architecture

- Marketplace catalog: `.agents/plugins/marketplace.json`
- Installable plugin root: `plugins/o-brain/`
- Plugin manifests: `plugins/o-brain/.codex-plugin/plugin.json`, `plugins/o-brain/.mcp.json`
- Hooks: `plugins/o-brain/hooks/memory-inject-hook.mjs`, `plugins/o-brain/hooks/memory-extract-hook.mjs`
- Skills: `plugins/o-brain/skills/<command>/` (original command names), plus `o-brain-*` compatibility skills
- Runtime launchers: `plugins/o-brain/scripts/`
- Development app and tests: `app/src/` and `app/web/`
- Packaged app copy: `plugins/o-brain/app/` (must match the tracked development app files)
- Default plugin data: `%LOCALAPPDATA%\SoDamAI\O-Brain\data` on Windows

## Invariants

- Bind HTTP only to `127.0.0.1`.
- Redact secrets before extraction or storage.
- Never commit DB, backups, env files, transcripts, model caches, or personal memory.
- Preserve both Codex and Claude transcript parsing unless a migration explicitly removes compatibility.
- Do not infer relation types automatically; require user confirmation.
- Back up before destructive DB changes.
- Keep installed-plugin data outside the plugin cache.
- Do not add a target GitHub URL until the repository owner provides or creates it.

## Required verification

From `app/`:

```powershell
npm ci
npm test
npm audit --omit=dev
```

Also run the plugin validator, `node --check` for project JS, JSON parsing, secret/tracked-file scans, and a real local `codex plugin add` flow before release claims.

Before packaging, run `node scripts/check-plugin-package.mjs`. Use `node scripts/sync-plugin-package.mjs` only when intentionally copying the tracked development app into the installable plugin package.

---
> Source: [sodam-ai/SoDam_O-Brain-Codex](https://github.com/sodam-ai/SoDam_O-Brain-Codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
