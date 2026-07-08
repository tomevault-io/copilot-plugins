## ideonomy-skill

> Guidelines for AI agents working in this repository.

# AGENTS.md

Guidelines for AI agents working in this repository.

## Repository Overview

This repository contains **Agent Skills** for AI agents following the [Agent Skills specification](https://agentskills.io/specification.md). Skills install to `.agents/skills/` (the cross-agent standard). This repo also serves as a **Claude Code plugin marketplace** via `.claude-plugin/marketplace.json`.

- **Name**: Ideonomy
- **GitHub**: [latentwill/ideonomy-skill](https://github.com/latentwill/ideonomy-skill)
- **Creator**: Ed Kennedy
- **License**: MIT

## Repository Structure

```
ideonomy-plugin/
├── .claude-plugin/
│   └── marketplace.json      # Claude Code plugin marketplace manifest
├── skills/
│   ├── ideonomy-plain/       # Plain-text-portable rendering
│   │   ├── SKILL.md
│   │   ├── bin/pick          # Random method-tuple picker
│   │   ├── methods/          # operators/, organons/, dimension-prompts/
│   │   └── examples/
│   └── ideonomy-rich/        # Monospace ASCII art rendering
│       ├── SKILL.md
│       ├── bin/pick
│       ├── methods/
│       └── rendering/        # Per-organon ASCII recipes
├── AGENTS.md
├── LICENSE
└── README.md
```

## Agent Skills Specification

### Required Frontmatter

```yaml
---
name: skill-name
description: What this skill does and when to use it.
---
```

### Name Field Rules

- Lowercase letters, numbers, and hyphens only
- Must match parent directory name exactly

## Claude Code Plugin

Install with:

```bash
/plugin marketplace add latentwill/ideonomy-skill
/plugin install ideonomy
```

See [Claude Code plugins documentation](https://code.claude.com/docs/en/plugins.md) for details.

## Adding Methods to the Catalog

The three method categories live under `skills/ideonomy-plain/methods/` and `skills/ideonomy-rich/methods/`. Both must stay in sync — changes to one skill's methods should be mirrored to the other.

Adding a new method: copy the file to both `skills/ideonomy-plain/methods/<category>/` and `skills/ideonomy-rich/methods/<category>/`. The `bin/pick` script discovers methods dynamically via `find`, so no manifest update is needed.

## Git Workflow

- New methods: `feature/<category>-<name>` (e.g., `feature/organon-flowchart`)
- Skill improvements: `fix/ideonomy-plain-description` or `fix/ideonomy-rich-rendering`
- Documentation: `docs/description`

Commit format: `feat: add <name> organon` / `fix: improve pick script mtime weighting` / `docs: update README`

---
> Source: [latentwill/ideonomy-skill](https://github.com/latentwill/ideonomy-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
