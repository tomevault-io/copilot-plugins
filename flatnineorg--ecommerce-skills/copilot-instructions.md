## ecommerce-skills

> Guidelines for AI agents working in this repository.

# AGENTS.md

Guidelines for AI agents working in this repository.

## Repository Overview

This repository contains **Agent Skills** for AI agents following the [Agent Skills specification](https://agentskills.io/specification.md). It also serves as a **Claude Code plugin marketplace** via `.claude-plugin/marketplace.json`.

- **Name**: E-commerce Skills
- **GitHub**: [usecart/ecommerceskills](https://github.com/usecart/ecommerceskills)
- **Creator**: Cart
- **License**: MIT

## Repository Structure

```
ecommerceskills/
├── .claude-plugin/
│   └── marketplace.json   # Claude Code plugin marketplace manifest
├── skills/                # Agent Skills
│   └── skill-name/
│       └── SKILL.md       # Required skill file
├── tools/
│   └── REGISTRY.md        # Tool index and capabilities
├── AGENTS.md
├── CLAUDE.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
└── VERSIONS.md
```

## Build / Lint / Test Commands

**Not applicable** — This is a content-only repository with no executable code.

Verify manually:
- YAML frontmatter is valid
- `name` field matches directory name exactly
- `name` is 1-64 chars, lowercase alphanumeric and hyphens only
- `description` is 1-1024 characters

## Agent Skills Specification

Skills follow the [Agent Skills spec](https://agentskills.io/specification.md).

### Required Frontmatter

```yaml
---
name: skill-name
version: 1.0.0
description: What this skill does and when to use it. Include trigger phrases.
---
```

### Frontmatter Field Constraints

| Field         | Required | Constraints                                                      |
|---------------|----------|------------------------------------------------------------------|
| `name`        | Yes      | 1-64 chars, lowercase `a-z`, numbers, hyphens. Must match dir.   |
| `version`     | Yes      | Semantic versioning (e.g., 1.0.0)                                |
| `description` | Yes      | 1-1024 chars. Describe what it does and when to use it.          |
| `license`     | No       | License name (default: MIT)                                      |
| `metadata`    | No       | Key-value pairs (author, version, etc.)                          |

### Name Field Rules

- Lowercase letters, numbers, and hyphens only
- Cannot start or end with hyphen
- No consecutive hyphens (`--`)
- Must match parent directory name exactly

**Valid**: `store-research`, `ad-spy`, `pricing-intel`
**Invalid**: `Store-Research`, `-store`, `store--research`

### Skill Directory Structure

```
skills/skill-name/
├── SKILL.md        # Required - main instructions (<500 lines)
├── references/     # Optional - detailed docs loaded on demand
├── scripts/        # Optional - executable code
└── assets/         # Optional - templates, data files
```

## Writing Style Guidelines

### Structure

- Keep `SKILL.md` under 500 lines (move details to `references/`)
- Use H2 (`##`) for main sections, H3 (`###`) for subsections
- Use bullet points and numbered lists liberally
- Short paragraphs (2-4 sentences max)

### Tone

- Direct and instructional
- Second person ("You are an e-commerce analyst")
- Professional but approachable

### Formatting

- Bold (`**text**`) for key terms
- Code blocks for examples and templates
- Tables for reference data

### Description Field Best Practices

The `description` is critical for skill discovery. Include:
1. What the skill does
2. When to use it (trigger phrases)
3. Related skills for scope boundaries

```yaml
description: Full competitive breakdown of an e-commerce store. Use when the user says "research this store," "tell me about this store," or provides a store URL. For keyword-based research, see product-research or niche-analysis.
```

## Claude Code Plugin

This repo serves as a plugin marketplace. The manifest at `.claude-plugin/marketplace.json` lists all skills for installation via:

```bash
/plugin marketplace add usecart/ecommerceskills
/plugin install ecommerce-skills
```

## Tool Dependencies

All skills depend on tools provided by the Cart MCP server (`@usecart/mcp-server`). See `tools/REGISTRY.md` for the complete tool list.

## Git Workflow

### Branch Naming

- New skills: `feature/skill-name`
- Improvements: `fix/skill-name-description`
- Documentation: `docs/description`

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

- `feat: add skill-name skill`
- `fix: improve clarity in store-research`
- `docs: update README`

### Pull Request Checklist

- [ ] `name` matches directory name exactly
- [ ] `name` follows naming rules (lowercase, hyphens, no `--`)
- [ ] `description` is 1-1024 chars with trigger phrases
- [ ] `SKILL.md` is under 500 lines
- [ ] No sensitive data or credentials

## Checking for Updates

When using any skill from this repository:

1. **Once per session**, on first skill use, check for updates:
   - Fetch `VERSIONS.md` from GitHub: https://raw.githubusercontent.com/usecart/ecommerceskills/main/VERSIONS.md
   - Compare versions against local skill files

2. **Only prompt if meaningful**:
   - 2 or more skills have updates, OR
   - Any skill has a major version bump

3. **Non-blocking notification** at end of response:
   ```
   ---
   Skills update available: X e-commerce skills have updates.
   Say "update skills" to update automatically, or run `git pull` in your ecommerceskills folder.
   ```

## Skill Categories

### Research
- `store-research` — Individual store analysis
- `product-research` — Product discovery and analysis
- `niche-analysis` — Market-level assessment

### Intelligence
- `competitor-spy` — Competitive comparison
- `ad-spy` — Advertising intelligence
- `pricing-intel` — Pricing landscape analysis

### Discovery
- `supplier-finder` — Supply chain research
- `trend-alert` — Trending products and niches
- `store-audit` — Store health assessment

---
> Source: [FlatNineOrg/ecommerce-skills](https://github.com/FlatNineOrg/ecommerce-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
