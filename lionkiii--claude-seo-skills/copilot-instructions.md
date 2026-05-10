## claude-seo-skills

> 44-command SEO skill system that runs inside Claude Code via `/seo <command>`.

# Claude SEO Skills

44-command SEO skill system that runs inside Claude Code via `/seo <command>`.

## Structure

- `skills/` — Skill definitions (SKILL.md + supporting files per command)
- `agents/` — Agent definitions for parallel audit delegation
- `scripts/` — Install, smoke-test, and verification scripts
- `skills/seo/` — Shared references, hooks, and Python scripts

## Install

```bash
bash install.sh
```

Copies skills/agents to `~/.claude/`, creates Python venv, verifies MCP registration.

## MCP Dependencies

- **Ahrefs MCP** — Connected via Claude.ai account (no local setup)
- **GSC MCP** (optional) — Bring-your-own Google Search Console MCP server, registered at user scope in `~/.claude/mcp.json`

## Dev Commands

- `bash install.sh --dry-run` — Preview install without changes
- `bash scripts/smoke-test.sh` — Validate all skills and agents
- `bash scripts/verify-mcp-scope.sh` — Check MCP registration status

---
> Source: [lionkiii/claude-seo-skills](https://github.com/lionkiii/claude-seo-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
