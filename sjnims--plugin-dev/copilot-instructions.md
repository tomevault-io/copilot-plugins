## plugin-dev

> Guidance for Claude Code working in this repository.

# CLAUDE.md

Guidance for Claude Code working in this repository.

## What This Is

Plugin marketplace containing the **plugin-dev** plugin - a toolkit for developing Claude Code plugins. Provides 10 skills, 3 agents, 4 slash commands.

**Version**: v0.4.0 | [CHANGELOG.md](CHANGELOG.md)

## MCP Tool Requirements (CRITICAL)

**Cost tiers**: FREE (Serena, rg) → PAID (Morph ~$0.8-1.2/1M tokens) → AVOID (built-in Grep/Edit)

### Search (prefer in order)

| Tool                              | Cost | Use When                               |
| --------------------------------- | ---- | -------------------------------------- |
| Serena `find_symbol`              | FREE | Know the symbol name                   |
| Serena `find_referencing_symbols` | FREE | Find all usages of a symbol            |
| Serena `get_symbols_overview`     | FREE | Understand file structure              |
| `rg "pattern"`                    | FREE | Regex/text patterns (not symbol-based) |
| Morph `warpgrep_codebase_search`  | PAID | Semantic/fuzzy queries (last resort)   |

### Edit (prefer in order)

| Tool                         | Cost  | Use When                           |
| ---------------------------- | ----- | ---------------------------------- |
| Serena `replace_symbol_body` | FREE  | Replacing entire methods/functions |
| Serena `insert_after_symbol` | FREE  | Adding new code after a symbol     |
| Morph `edit_file`            | PAID  | Partial edits, non-LSP files       |
| Built-in `Edit`              | AVOID | Fallback only                      |

> **FREE tools first. Morph costs real money. Built-in Edit only as last resort.**

## Critical: Two-Level Architecture

This repo has TWO `.claude-plugin/` directories:

| Level  | Path                                             | Purpose              |
| ------ | ------------------------------------------------ | -------------------- |
| Root   | `/.claude-plugin/marketplace.json`               | Marketplace manifest |
| Plugin | `/plugins/plugin-dev/.claude-plugin/plugin.json` | Plugin manifest      |

**Testing**: `claude --plugin-dir plugins/plugin-dev` (NOT root)

## Essential Commands

```bash
# Test plugin locally
claude --plugin-dir plugins/plugin-dev

# Lint markdown
markdownlint-cli2 '**/*.md'

# Format markdown
prettier --write '**/*.md'

# Lint shell scripts
shellcheck plugins/plugin-dev/skills/*/scripts/*.sh

# Lint YAML files
uvx yamllint .github/workflows/
```

## Key Conventions

- **Paths**: Use `${CLAUDE_PLUGIN_ROOT}` for portable paths
- **Skills**: Progressive disclosure (SKILL.md + references/ + examples/)
- **Descriptions**: Third-person ("This skill should be used when...")
- **Versions**: Sync across plugin.json, marketplace.json, CLAUDE.md

## On-Demand Documentation

| Topic              | File                                                     |
| ------------------ | -------------------------------------------------------- |
| Version releases   | [docs/release-procedure.md](docs/release-procedure.md)   |
| Component patterns | [docs/component-patterns.md](docs/component-patterns.md) |
| CI/CD workflows    | [docs/ci-cd.md](docs/ci-cd.md)                           |
| Troubleshooting    | [docs/troubleshooting.md](docs/troubleshooting.md)       |
| Workflow security  | [docs/workflow-security.md](docs/workflow-security.md)   |

## Quick Fixes

| Problem              | Solution                           |
| -------------------- | ---------------------------------- |
| Plugin not loading   | Use `plugins/plugin-dev`, not root |
| Skill not triggering | Add trigger phrases to description |
| Validation fails     | Run component validator script     |

## Validation Agents

- **plugin-validator**: Validates plugin structure and manifests
- **skill-reviewer**: Reviews skill quality and triggering
- **agent-creator**: Generates agents from descriptions

---
> Source: [sjnims/plugin-dev](https://github.com/sjnims/plugin-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
