## research-junshi

> Both hosts load the root `SKILL.md`; there is one research workflow and one memory format.

# Agent adapters

Both hosts load the root `SKILL.md`; there is one research workflow and one memory format.

| Host | Install directory | Invoke | Tool mapping |
|---|---|---|---|
| Claude Code | `~/.claude/skills/research-junshi` | `/research-junshi` or “Run research-junshi” | Read/PDF tools, Bash for shared Python scripts, WebSearch/WebFetch when available |
| Codex | `~/.agents/skills/research-junshi` | `$research-junshi` | Available file/PDF tools, terminal for shared Python scripts, available web search/browser tools |
| Other Agent Skills hosts | Host's skill directory | Host's skill invocation | Map file, terminal, and web capabilities to the same workflow |

Codex's UI metadata is `agents/openai.yaml`. Installation locations follow the [official Codex skill documentation](https://learn.chatgpt.com/docs/build-skills). Claude's [skill documentation](https://code.claude.com/docs/en/skills) describes its user skill directory.

Require Python 3.9+ for the core. If shell/Python or browsing is unavailable, report which stage cannot run; do not pretend to have persisted memory or searched sources. Codex installations may require approval to write outside the workspace to `~/.junshi`; use normal permissions, or set `JUNSHI_HOME` to a private writable directory and use that same path in all hosts and cron. Do not create an unnoticed second memory store to avoid a permission prompt.

No host CLI is required by `daily.py`: scheduling is agent-independent. Future adapters should only map tools and installation, preserving the shared memory commands, identity rules, and idea-scoring formula.

---
> Source: [junshi-research/research-junshi](https://github.com/junshi-research/research-junshi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
