## rebana-skills

> The complete collection of Kilo Code configs: agents, skills, commands, rules, and

# Everything Kilo Code

The complete collection of Kilo Code configs: agents, skills, commands, rules, and
MCP configurations for production-grade AI-assisted development.

This is a Kilo Code port of `everything-claude-code` (Anthropic hackathon winner). It is
loaded automatically by Kilo from this directory: agents in `.kilo/agent/`, commands in
`.kilo/command/`, skills in `.kilo/skill/`, and the rule files in `rules/` (via the
`instructions` glob in `kilo.json`).

## How to use

- **Commands** run with `/<name>` (e.g. `/tdd`, `/plan`, `/code-review`, `/verify`).
- **Agents** are subagents invoked by the model when delegated (e.g. `code-reviewer`,
  `security-reviewer`, `planner`, `architect`, `tdd-guide`).
- **Skills** are loaded automatically from `.kilo/skill/`.
- **Rules** in `rules/` are always-follow guidelines (security, coding-style, testing,
  git-workflow, agents, performance, patterns, hooks).

## Layout

```
everything-kilo-code/
|-- kilo.json            # MCP servers, permissions, instructions, skills paths
|-- AGENTS.md            # This file (project instructions)
|-- .kilo/
|   |-- agent/           # Subagents (mode: subagent)
|   |-- command/         # Slash commands
|   |-- skill/           # 98 curated general-purpose skills (SKILL.md each)
|-- agents/              # Source (Claude Code) agents, mirrored to .kilo/agent
|-- commands/            # Source commands, mirrored to .kilo/command
|-- skills/              # Curated toolkit skills (subset also in .kilo/skill)
|-- rules/               # Always-follow guidelines (loaded by instructions glob)
|-- hooks/               # Session/memory hooks (reference; see notes below)
|-- scripts/             # Cross-platform Node.js tooling
|-- tests/               # Node test suite (node tests/run-all.js)
|-- mcp-configs/         # MCP server definitions
|-- examples/            # Example CLAUDE.md / session memory files
|-- contexts/            # System-prompt injection contexts
```

## Regenerating `.kilo/`

The `.kilo/agent/` and `.kilo/command/` directories are generated from `agents/` and
`commands/` by `scripts/kilo-convert.js`. Re-run it after editing the source files:

```bash
node scripts/kilo-convert.js
```

## Notes on hooks

Kilo Code's config (`kilo.json`) does not expose the Claude Code hooks system. The
scripts under `hooks/` and `scripts/hooks/` (memory persistence, strategic compaction,
continuous learning) are kept for reference and for setups that support hook execution.
Session context is best preserved using Kilo's built-in memory/session tools.

## Customization

Start with what resonates, modify for your stack, remove what you don't use, add your own
patterns. Agent `model:` fields use short names (`opus`/`sonnet`/`haiku`); set them to
models available on your provider if you want explicit routing.

## Skill Curation

This repo ships **68 general-purpose skills** selected to work with any LLM provider
(StepFun, Anthropic, OpenAI, Google, etc.). Vendor/framework-specific skills
(Angular, dbt, Vercel, Figma, Slack, YouTube, Databricks, ClickHouse, LangSmith, dll)
are intentionally excluded to keep the toolkit lean and model-agnostic.

Skills are sourced from the built-in Kilo catalog, plus curated additions from
[ConardLi/garden-skills](https://github.com/ConardLi/garden-skills),
[VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents),
[mturac/everything-openai-codex](https://github.com/mturac/everything-openai-codex),
[faizkhairi/claude-code-blueprint](https://github.com/faizkhairi/claude-code-blueprint),
[ciembor/agent-rules-books](https://github.com/ciembor/agent-rules-books),
[williamkho77/claude-humanizer-id](https://github.com/williamkho77/claude-humanizer-id),
[ariosenodev/humanizer-bahasa-indonesia](https://github.com/ariosenodev/humanizer-bahasa-indonesia),
[Mindrally/skills](https://github.com/Mindrally/skills),
[gamedev-skills/awesome-gamedev-agent-skills](https://github.com/gamedev-skills/awesome-gamedev-agent-skills),
[ImL1s/flutter-claude-skills](https://github.com/ImL1s/flutter-claude-skills),
[dart-lang/skills](https://github.com/dart-lang/skills), and
[flutter/agent-plugins](https://github.com/flutter/agent-plugins).

## MCP Servers

`kilo.json` includes preconfigured MCP servers for common services:
GitHub, Firecrawl, Supabase, Memory, Sequential Thinking, Vercel, Railway,
Cloudflare (Docs / Workers Builds / Workers Bindings / Observability),
ClickHouse, Context7, Magic, Filesystem, Brave Search, Postgres, Redis, Docker,
Fetch, and Playwright.

Set the required environment variables before use (e.g. `GITHUB_PERSONAL_ACCESS_TOKEN`,
`FIRECRAWL_API_KEY`, `BRAVE_API_KEY`). Disable any server by setting `"enabled": false`.

## Token Saver

Experimental token-saving toggles are exposed under `experimental` in `kilo.json`:
- `token_saver_rtk` — compress tool output like `git`, `grep`, `ls`, `tree`, `logs`
- `token_saver_headroom` — compress prompts via `/v1/compress` before routing to the model
- `token_saver_caveman` — terse-style system prompt for ~65% fewer output tokens
- `token_saver_ponytail` — lazy senior dev bias: YAGNI, reuse stdlib, deletion over addition

These map to community tools RTK, Headroom, Caveman, and Ponytail. Enable only the ones
you actually install; leaving them enabled without the corresponding tool/hook has no effect.

## Recommended Config

Useful defaults are also enabled at the top level of `kilo.json`:
- `web_search: true` — make web search available to all models
- `default_agent: "build"` — default primary agent when none is specified
- `auto_collapse_reasoning: true` — collapse reasoning blocks after completion
- `terminal_command_display: "collapsed"` — start terminal command blocks collapsed in UI

---
> Source: [RebanaDev/Rebana-Skills](https://github.com/RebanaDev/Rebana-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
