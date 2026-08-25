## ua-skills

> This repo (`appeeky/ua-skills`) contains mobile app user acquisition skills — ads, creatives, and ROAS — following the [Agent Skills](https://agentskills.io) open standard.

# Agent Guidelines

This repo (`appeeky/ua-skills`) contains mobile app user acquisition skills — ads, creatives, and ROAS — following the [Agent Skills](https://agentskills.io) open standard.

## Structure

```
skills/<name>/SKILL.md     — Skill instructions (required, <500 lines)
skills/<name>/references/  — Detailed docs loaded on demand (optional)
tools/integrations/        — Appeeky API integration guides
tools/REGISTRY.md          — Capability matrix: which skill uses which tool
```

Compatible directories: `.cursor/skills/`, `.agents/skills/`, `.claude/skills/`, `.codex/skills/`

## SKILL.md Format

```yaml
---
name: skill-name          # Must match directory, lowercase+hyphens, 1-64 chars
description: ...          # Trigger phrases + scope boundaries, 1-1024 chars
metadata:
  version: 1.0.0
---
```

Description drives discovery — agent reads all descriptions to decide which skill to load. Include "When the user wants to...", trigger keywords, and "For X, see Y skill" boundaries.

## Writing Style

- Direct, actionable, second person
- Tables for comparisons, numbered lists for steps
- Bold for key terms, code blocks for examples
- No filler, no hedging
- Be honest about ad economics: CPI/CPA, margin, creative fatigue, learning spend

## Appeeky Integration

Skills work standalone with frameworks. Connect [Appeeky MCP](https://docs.appeeky.com/docs/mcp) for live data and campaign operations.

MCP tool arguments are **snake_case** (`app_url`, `campaign_id`, `job_id`). REST JSON bodies are **camelCase** (`appUrl`, `jobId`). Job/result fields from the API stay camelCase (`result.ad.imageUrl`).

Key integrations: App Ad Creatives, Meta Marketing API, TikTok Marketing API, Apple Search Ads, RevenueCat.

## Related Repo

For organic ASO and App Store listing optimization, see [aso-skills](https://github.com/eronred/aso-skills). Pairing: `aso-skills` = organic, `ua-skills` = paid.

## Commits

`feat(skill-name): ...` / `fix(skill-name): ...` / `docs: ...`

Branches: `feature/skill-name`, `fix/skill-name-desc`, `docs/desc`

---
> Source: [appeeky/ua-skills](https://github.com/appeeky/ua-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
