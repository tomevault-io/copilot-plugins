## webflow-skills

> Webflow Skills is a marketplace-ready collection of agent skills for Claude Code, Cursor, and compatible agents. Skills extend agents with specialized Webflow capabilities: CMS management, site auditing, safe publishing, code components, and CLI integration. **This is a pure documentation project** — no compiled code, just SKILL.md files that guide agent behavior.

## Project Overview

Webflow Skills is a marketplace-ready collection of agent skills for Claude Code, Cursor, and compatible agents. Skills extend agents with specialized Webflow capabilities: CMS management, site auditing, safe publishing, code components, and CLI integration. **This is a pure documentation project** — no compiled code, just SKILL.md files that guide agent behavior.

## Repository Structure

Three plugin families live under `plugins/`:

- **webflow-skills/** — CMS operations, site/asset/link audits, safe publish, FlowKit naming (10 skills)
- **webflow-code-component-skills/** — React code component scaffolding, conversion, validation, deployment (7 skills)
- **webflow-cli-skills/** — CLI wrappers for Cloud, DevLink, designer extensions (5 skills)

Key config files:

- `.mcp.json` — MCP server config (Webflow MCP at https://mcp.webflow.com/mcp)
- `.claude-plugin/marketplace.json` — Marketplace entry listing all 3 plugins
- `.cursor-plugin/plugin.json` + `marketplace.json` — Cursor plugin config
- `agents/webflow-agent.md` — Agent setup documentation

## CI Validation

CI runs on push/PR to main (`.github/workflows/ci.yml`):

```bash
# What CI checks:
# 1. Required root files: README.md, LICENSE, AGENTS.md, CLAUDE.md (must be symlink)
# 2. Marketplace and plugin configs exist
# 3. Every skill dir under plugins/webflow-skills/skills/ has SKILL.md with name: and description: frontmatter
```

There is no build step, test suite, or linter config — validation is purely structural.

## Skill Anatomy

Each skill is a single `SKILL.md` file in its own directory:

```
plugins/<plugin-name>/skills/<skill-name>/SKILL.md
```

Required YAML frontmatter:

```yaml
---
name: skill-name # 1-64 chars, lowercase alphanumeric + hyphens
description: ... # Up to 1024 chars, include trigger keywords
---
```

Optional frontmatter: `license`, `model`, `allowed-tools`

## Design Conventions

- **Phased workflows**: Discovery → Analysis → Validation → Preview → Confirmation → Execution → Reporting
- **Safety-first**: All mutating operations require explicit user confirmation (type "confirm" or "publish", not just "yes")
- **Batch operations**: Large datasets split into 50-item batches with progress reporting
- **MCP-routed**: Webflow operations use MCP tools, never direct API calls

## Adding/Editing Skills

Follow `CONTRIBUTING.md`. PR title format: `feat(skills): Add [skill-name]`. Keep SKILL.md under 500 lines. Every skill needs Instructions, Examples, and Guidelines sections.

---
> Source: [webflow/webflow-skills](https://github.com/webflow/webflow-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
