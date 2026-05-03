## drug-discovery-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **pharmaceutical drug discovery intelligence** plugin for Claude Code. It provides specialized skills for target analysis, competitive intelligence, literature review, and clinical trial analysis.

## Architecture

The project follows Claude Code's official Skill structure:

```
.claude/
└── skills/
    ├── target-profile/           # Target dossier generation
    │   ├── SKILL.md              # Main skill documentation
    │   ├── reference/            # Detailed API docs, field specs, examples
    │   └── scripts/               # Data fetching utilities
    ├── competitor-analysis/     # Competitive intelligence
    ├── literature-review/        # Literature search and summarization
    └── clinical-intelligence/    # Clinical trials analysis
```

### Skill Structure Pattern

Each skill follows the **progressive disclosure** pattern:
- **SKILL.md** - Lightweight instruction file (<500 lines) with YAML frontmatter (`name`, `description`)
- **reference/** - Detailed documentation loaded only when needed (API docs, field definitions, examples)
- **scripts/** - Utility scripts that execute without loading their contents into context

### Skill Activation

Skills are **model-invoked**: Claude automatically selects relevant skills based on user request matching the skill's `description`. No explicit slash commands needed.

## Available Skills

| Skill | Trigger Keywords | Output |
|-------|------------------|--------|
| `target-profile` | target, dossier, validation, druggability | Target assessment report |
| `competitor-analysis` | competitor, competition, pipeline, landscape | Competitive intelligence report |
| `literature-review` | literature, papers, pubmed, search | Literature summary with trends |
| `clinical-intelligence` | clinical, trials, enrollment, study | Trial analysis and predictions |

## Data Sources

- **Open Targets Platform** - Disease associations, tractability
- **UniProt** - Protein annotation
- **ChEMBL** - Bioactive compounds
- **ClinicalTrials.gov** - Clinical trials
- **PubMed** - Scientific literature
- **USPTO/EPO** - Patent data

## Running Scripts

The `scripts/` directories contain Python utilities for data fetching:

```bash
# Fetch target data from Open Targets, UniProt, ChEMBL
python .claude/skills/target-profile/scripts/fetch_data.py EGFR --output data.json

# Include all sources
python .claude/skills/target-profile/scripts/fetch_data.py EGFR --uniprot --chembl -o full.json
```

Requirements: `pip install requests`

## Adding New Skills

1. Create directory: `.claude/skills/your-skill/`
2. Create `SKILL.md` with YAML frontmatter (`name`, `description`)
3. Add `reference/` for detailed documentation
4. Add `scripts/` for utilities (optional)

**SKILL.md template:**

```yaml
---
name: your-skill-name
description: Brief description of what this skill does and when to use it. Include trigger keywords.
---

# Skill Name

Quick overview of the skill.

## Quick Start

```
/example-usage
```

## What's Included

| Section | Description |
|---------|-------------|

## Data Sources

## Requirements

## Additional Resources

- See [reference/api.md](reference/api.md) for details
```

## Key Design Principles

1. **SKILL.md stays concise** - Put detailed content in `reference/` files
2. **Description matters** - Claude matches requests against `description` field, include specific trigger keywords
3. **Scripts execute, don't load** - Scripts run without consuming context tokens
4. **Public APIs first** - Prioritize free/public data sources over commercial ones

---
> Source: [huifer/drug-discovery-skills](https://github.com/huifer/drug-discovery-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
