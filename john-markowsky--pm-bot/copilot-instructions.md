## core-workflow

> Essential workflow patterns for NEXUS v2 development


# 🚀 NEXUS v2 Core Workflow

## Mode Framework (MANDATORY)
- **PLAN Mode**: `# Mode: PLAN` - Analysis & planning only, no code changes
- **ACT Mode**: `# Mode: ACT` - Execute approved plans, build & test
- **RCA Mode**: `# Mode: RCA` - Root cause analysis only, no fixes

## Safety Rules (MANDATORY)
- Never run destructive commands without user approval
- Always ask before git operations or deployments
- Use containerized commands: `docker exec service command`
- No direct host commands (npm, python, etc.)

## Mode Loading System
- **When user invokes mode**: Load `.cursor/modes/{mode}.md` and follow it
- **PLAN loads**: `.cursor/modes/plan.md`
- **ACT loads**: `.cursor/modes/act.md`
- **RCA loads**: `.cursor/modes/rca.md`

## Reference Loading System
- **MCP Cheat Sheet**: Load `.cursor/rules/mcp-cheat-sheet.mdc` when needed
- **Multi-Agent**: Load `.cursor/rules/multi-agent-coordination.mdc` when needed
- **Sprint Planning**: Load `.cursor/rules/sprint-planning.mdc` when needed
- **Production VM**: Load `.cursor/rules/production-vm-workflow.mdc` when needed

## MCP Server Priority (Top 3)
1. **mcp_repo-indexer_*** (code search, complexity, dependencies)
2. **mcp_loki_*** (logs - 100x faster than docker logs)
3. **Browser Agent** (frontend testing, no Playwright)

## Output Contract
- **Default**: Patches only, minimal explanation
- **When requested**: Include reasoning and context
- **Always**: Confirm before destructive operations

---
> Source: [john-markowsky/PM-Bot](https://github.com/john-markowsky/PM-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
