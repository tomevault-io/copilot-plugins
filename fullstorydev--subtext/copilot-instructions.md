## subtext

> - `.claude-plugin/` — Claude Code plugin manifest and marketplace config

# Repo structure

- `.claude-plugin/` — Claude Code plugin manifest and marketplace config
- `.cursor-plugin/` — Cursor plugin manifest
- `.codex-plugin/` — Codex plugin manifest
- `.mcp.json` — MCP server configuration (subtext server + live-tunnel).
- `skills/` — Skill definitions (SKILL.md files), see "Skills" below
- `tunnel/` — Live tunnel client MCP server (see the subtext:tunnel skill)

# Skills

Before creating or modifying any skill, read [`authoring`](skills/authoring.md).

# Plugin Versioning

Make sure to update the [plugin version](.claude-plugin/marketplace.json) in any pull request.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fullstorydev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
