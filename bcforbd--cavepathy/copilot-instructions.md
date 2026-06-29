## cavepathy

> These rules apply regardless of which agent platform runs the skill:

# Multi-Agent Behavioral Guidelines for Cavepathy

## Universal rules (all agents)

These rules apply regardless of which agent platform runs the skill:

1. **Assumption surfacing**: Before writing code, state what you're assuming about scope, algorithm, data format, or environment. If multiple interpretations exist, present them.

2. **Minimal implementation**: Write the smallest code that solves the problem. No speculative features, no unnecessary abstractions, no "flexibility" that wasn't requested.

3. **Surgical edits**: Touch only what the user asked to change. Don't refactor adjacent code, don't "improve" comments, don't reformat unrelated lines.

4. **Goal-driven loop**: Define what "done" looks like before starting. Verify before claiming completion. Don't narrate steps - execute them.

5. **Compressed output**: Drop filler, articles, hedging, pleasantries. Keep all technical substance exact. Code blocks and error strings stay verbatim.

## Agent-specific adaptations

### Claude Code
- Skill lives in `.claude/skills/cavepathy/`
- Use `/cavepathy [level]` slash command
- Compression applies to all conversational output
- Hook: inject via `UserPromptSubmit` for session persistence

### OpenAI Codex / ChatGPT
- Skill discovered via `skills/cavepathy/agents/openai.yaml`
- Implicit invocation on triggers: "caveman", "cavepathy", "be concise"
- Show in UI picker under "coding-style" category

### Hermes Agent
- Skill discovered via `skills/cavepathy/agents/hermes.yaml`
- Requires tools: read, write, edit, grep, glob, bash, agent
- Platform-specific: hermes-agent, hermes-cli, hermes-web

### OpenClaw
- Drop `cavepathy/` folder into `~/.openclaw/workspace/skills/`
- `SOUL.md` at repo root provides persona context
- Skill body loaded as "Project Context"

### Gemini CLI / Kilo Code / OpenCode
- Scan `.agents/skills/` for discovery
- Place skill folder at `.agents/skills/cavepathy/`
- SKILL.md frontmatter `name` must match directory name

### Cline
- Rules-based via `.clinerules/`
- Wrap scripts in `scripts/` as type-safe tools for SDK ingestion
- Template: `cline_mcp_settings.json` for MCP connection

### Kiro CLI
- Uses Agent Connection Protocol (ACP)
- Reference in parent agent routing rules
- Delegate compressed coding tasks to Kiro

## Installation paths

| Agent | Install path | Discovery |
|---|---|---|
| Claude Code | `~/.claude/skills/cavepathy/` | `.claude/skills/` |
| Codex | `~/.codex/skills/cavepathy/` | `.agents/skills/` |
| Hermes | `hermes skills install <url>` | `skills/cavepathy/agents/hermes.yaml` |
| OpenClaw | `~/.openclaw/workspace/skills/cavepathy/` | `SOUL.md` + skill body |
| Gemini CLI | `.agents/skills/cavepathy/` | `.agents/skills/` scan |
| Kilo Code | `.agents/skills/cavepathy/` | `.agents/skills/` scan |
| OpenCode | `.agents/skills/cavepathy/` | `.agents/skills/` scan |
| Cline | `.clinerules/` or MCP | `cline_mcp_settings.json` |

---
> Source: [bcforbd/cavepathy](https://github.com/bcforbd/cavepathy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
