## fluid-claude-skills

> This repository contains Claude Code skills for Fluid Commerce, an ecommerce platform for MLM and social selling companies.

# Fluid Skills — Agent Guidelines

This repository contains Claude Code skills for Fluid Commerce, an ecommerce platform for MLM and social selling companies.

## Repository Structure

```
skills/
  {skill-name}/
    SKILL.md          # Required — skill instructions
    references/       # Optional — supplementary docs, schemas, examples
```

## Skill File Conventions

- **Directory name** = kebab-case, must match the `name` field in frontmatter
- **SKILL.md** is the only required file per skill
- Keep SKILL.md under 500 lines — move detailed references to `references/`
- YAML frontmatter is required with `name`, `description`, and `metadata.version`

### Frontmatter format

```yaml
---
name: skill-name
description: >-
  When to use this skill. Include trigger phrases and keywords.
metadata:
  version: 1.0.0
---
```

- `name`: 1-64 chars, lowercase alphanumeric and hyphens only, no consecutive hyphens
- `description`: 1-1024 chars, should include trigger conditions starting with "Use when..."

## Fluid Platform Context

- **API base URL**: `https://{company}.fluid.app`
- **Auth**: `Authorization: Bearer <FLUID_API_KEY>` on all API requests
- **Primary API versions**: `/api/company/v1/` (products, categories) and `/api/v2/` (metafields, settings)
- **DAM**: Images are uploaded via ImageKit and registered through `/api/dam/assets/`
- **Country ID**: Default is 214 (US), resolved via `GET /api/settings/company_countries`

## Cross-Referencing Skills

Skills can reference each other. Use relative paths from the repo root:

```markdown
See [fluid-product-admin-import](skills/fluid-product-admin-import/SKILL.md) for product payload structure.
```

## Fluid API Key

A test API key is stored in `.env` as `FLUID_API_KEY`. Only use this when the user explicitly asks to interact with the Fluid API (e.g. onboarding, product import, theme operations). Do not load or reference it for other tasks.

## When Editing Skills

- Test that frontmatter parses as valid YAML
- Verify all API endpoints and field names against the current Fluid API
- Keep code examples minimal and correct — they will be used as templates
- Don't duplicate information across skills; cross-reference instead

---
> Source: [felipelee/fluid-claude-skills](https://github.com/felipelee/fluid-claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
