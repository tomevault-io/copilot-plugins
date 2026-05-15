## parcl-ai-connect

> This repository contains use cases and Claude Code skills for the Parcl Labs MCP (Model Context Protocol) server. It is **not** an MCP server itself. It is a collection of workflows that use the Parcl Labs MCP to solve real estate analysis problems.

# Parcl AI Connect

This repository contains use cases and Claude Code skills for the Parcl Labs MCP (Model Context Protocol) server. It is **not** an MCP server itself. It is a collection of workflows that use the Parcl Labs MCP to solve real estate analysis problems.

## Quick Start

1. Connect the Parcl Labs MCP server in Claude Code
2. Browse `use-cases/` for available workflows
3. **Basic**: Copy a prompt from any `use-cases/<name>/basic/PROMPT.md` and paste it into Claude Code
4. **Advanced**: Run skills directly with `/<skill-name> [arguments]`

## Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| Accidental Landlord | `/accidental-landlord [market]` | Lead gen pipeline for failed sellers who pivoted to renting |
| Competitor Analysis | `/competitor-analysis [investor]` | 8-tab portfolio teardown of any institutional SFR investor |
| PM Fragmentation | `/pm-fragmentation [market]` | PM consolidation opportunity scoring across portfolios |
| Property Manager Intel | `/property-manager-intel [market or 'national']` | Ranked PM list separating third-party PMs, brokerages, institutional landlords, and platforms |
| Acquisition Targeting | `/acquisition-targeting [markets]` | Disposition signal detection for bulk acquisition targets |
| Market Study | `/market-study [market]` | Institutional-grade PDF with KPI matrices and supply-demand analysis |
| Broker Analytics | `/broker-analytics [market]` | Agent/brokerage performance ranked by motivated seller distress signals |
| HVAC Direct Mail | `/hvac-direct-mail [market]` | Direct mail target list with route-optimized delivery zones for HVAC outreach |
| Pool Service Targeting | `/pool-service-targeting [market]` | New homeowner pool service leads with timing tiers and cluster routing |

## Repository Structure

```text
.claude/skills/          # Claude Code skills (advanced workflows)
use-cases/               # Use case documentation with basic prompts
  <name>/basic/PROMPT.md # Copy-paste prompt (no setup needed)
  <name>/README.md       # Use case overview
assets/                  # Videos, GIFs, branding
docs/                    # Contributing guide
```

## Conventions

- **File naming**: kebab-case for all directories and markdown files. Snake_case acceptable for data files (CSVs, images).
- **Skills**: Each skill lives in `.claude/skills/<skill-name>/SKILL.md` with supporting files underneath.
- **Prompts**: Basic prompts are self-contained and should work with zero setup beyond having the MCP connected.
- **Dashboards**: All HTML output is single-file with no external dependencies.
- **Data sources**: All data comes from the Parcl Labs MCP. The primary tools are documented in each skill's tool mapping table.
- **Human-readable names**: Never display `parcl_id` values in user-facing output (dashboards, reports, CSVs, workbooks). Always resolve parcl_ids to proper market names, ZIP codes, city names, or investor names before rendering. Note that some MCP endpoints return `parcl_id` and `msa_parcl_id` as floats with `.0` suffix (e.g., `5452730.0`). Strip the `.0` before lookups.

## Development

- Do not commit to `main` directly. Use feature branches and PRs.
- Run `npx markdownlint-cli2 '**/*.md'` to lint markdown before submitting.
- Skill files must have `name` and `description` in frontmatter. Description must be under 250 characters.

---
> Source: [ParclLabs/parcl-ai-connect](https://github.com/ParclLabs/parcl-ai-connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
