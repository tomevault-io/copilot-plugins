## claude-cloud-diagrams

> Generates architecture diagrams from infrastructure JSON.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Plugin Structure

This is a Claude Code plugin that provides two skills for AWS infrastructure visualization.

```
claude-cloud-diagrams/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   ├── aws-discover/        # Discovery skill
│   │   └── SKILL.md
│   └── aws-diagram/         # Diagram generation skill
│       └── SKILL.md
├── discover.py              # Standalone CLI script
├── generate_diagram.py      # Standalone CLI script
└── example/                 # Sample output
```

## Skills

### aws-discover
Discovers AWS infrastructure and saves to JSON.

**Trigger phrases:** "discover AWS", "explore AWS account", "scan infrastructure"

**Example:** "discover my AWS infrastructure using profile prod-account in us-east-1"

### aws-diagram
Generates architecture diagrams from infrastructure JSON.

**Trigger phrases:** "generate diagram", "create AWS diagram", "visualize infrastructure"

**Example:** "generate all AWS diagrams as SVG"

## Alternative: Python Scripts

For automation or CI/CD:

```bash
# Discover infrastructure
uv run discover.py --profile myprofile --region eu-west-1

# Generate diagrams
uv run generate_diagram.py all
uv run generate_diagram.py architecture --format svg
```

## Prerequisites

- Graphviz: `brew install graphviz`
- Python dependencies: `uv sync`
- AWS CLI configured: `aws sso login --profile <profile>`

---
> Source: [mpuig/claude-cloud-diagrams](https://github.com/mpuig/claude-cloud-diagrams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
