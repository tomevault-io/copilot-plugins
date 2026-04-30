## skrillz

> This file provides guidance to Claude Code when working in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Repository Overview

**Skrillz** is a Claude Code plugin containing **217 production skills** for autonomous coding, API integrations, cloud services, testing, and development workflows.

**Distribution**: Private GitHub plugin (install via `/plugin marketplace add`)

## Plugin Structure

```
skrillz/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata
│   └── marketplace.json     # Distribution config
├── skills/                  # 217 production skills
├── hooks/                   # Optional telemetry hooks
├── docs/                    # Reference documentation
├── scripts/                 # Utility scripts
├── skill-builder-package/   # Skill creation toolkit
├── BUILD-ROADMAP.md        # Development roadmap
└── SUPER-PLAN-VISION.md    # Ecosystem vision
```

## Skill Categories

| Prefix | Count | Purpose |
|--------|-------|---------|
| `ac-*` | 28 | Autonomous coding (TDD, state, orchestration) |
| `bedrock-*` | 14 | AWS Bedrock agents and patterns |
| `auto-claude-*` | 9 | Auto-Claude build system |
| `ralph-*` | 7 | Quality review loops |
| `autonomous-*` | 5 | Autonomous operation |
| `financial-*` | 12 | Financial data APIs |
| `testing-*` | 13 | Testing frameworks |
| Other | 129+ | Cloud, MCP, plugins, workflows |

## Key Skills

**Autonomous Coding**:
- `ac-master-controller` - Main orchestrator for autonomous workflows
- `ac-tdd-runner` - Test-driven development automation
- `ac-spec-generator` - Generate feature specifications
- `ac-parallel-coordinator` - Multi-agent coordination

**Quality & Review**:
- `ralph-wiggum-v2` - Iterative quality improvement loops
- `ralph-skill-review-loop` - Automated skill reviews

**Cloud & Infrastructure**:
- `bedrock-agents` - AWS Bedrock foundation model orchestration
- `eks-*` - Kubernetes on AWS patterns
- `terraform-*` - Infrastructure as code

## Skill Architecture

Skills follow **progressive disclosure**:

1. **SKILL.md** - Always loaded, contains overview and quick start
2. **references/** - Detailed guides, loaded on-demand
3. **scripts/** - Automation tools, loaded when needed

### YAML Frontmatter

Every SKILL.md starts with:
```yaml
---
name: skill-name-in-hyphen-case
description: [What it does]. Use when [triggers].
---
```

### Organizational Patterns

1. **Workflow-based**: Sequential steps (deployment, integration)
2. **Task-based**: Independent operations (troubleshooting)
3. **Reference**: Standards and guidelines (design systems)
4. **Capabilities-based**: Feature suites with multiple entry points

## Creating New Skills

Use the `skill-builder-package/`:

1. **Study examples** in `skill-builder-package/examples/`
2. **Read research** in `skill-builder-package/research/`
3. **Follow patterns** from `skill-builder-package/planning/`
4. **Use skill-builders** in `skill-builder-package/skill-builders/`

### Adding Skills

Place new skills in `skills/` following the patterns above.

## Environment

Copy example files for local development:
```bash
cp .env.example .env
cp .env.financial-apis.example .env.financial-apis
```

## Important Notes

- This is a **Claude Code plugin**, installable via `/plugin`
- Skills are in `skills/` directory (plugin standard location)
- All paths are relative for portability
- Can be installed from private GitHub with token

## Research Foundation

Built on:
- Anthropic official documentation
- 40+ community skills analyzed
- 10 community innovations identified
- 4 organizational patterns validated

---
> Source: [adaptationio/Skrillz](https://github.com/adaptationio/Skrillz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
