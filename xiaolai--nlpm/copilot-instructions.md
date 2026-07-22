## nlpm

> This Codex plugin ships nlpm's **reference-knowledge skills**: the 50 Rules of Natural Language Programming, the 100-point scoring rubric, per-tool conventions (Claude Code / Codex CLI / Antigravity), anti-patterns, the vocabulary discipline framework, and authoring guides for skills, agents, hooks, rules, plugins, and prompts.

# nlpm — reference knowledge for Codex CLI

This Codex plugin ships nlpm's **reference-knowledge skills**: the 50 Rules of Natural Language Programming, the 100-point scoring rubric, per-tool conventions (Claude Code / Codex CLI / Antigravity), anti-patterns, the vocabulary discipline framework, and authoring guides for skills, agents, hooks, rules, plugins, and prompts.

These skills are knowledge, not commands. Load one when you need it — e.g. ask Codex to "score this skill against the nlpm rules" and it pulls in `$nlpm-rules` and `$nlpm-scoring`; ask "what's the Codex skill layout" and it pulls `$nlpm-conventions-codex`.

## What this plugin does NOT include

nlpm's interactive linting commands (`/nlpm:score`, `/nlpm:check`, `/nlpm:fix`, etc.) are **not** ported to Codex. Those commands orchestrate sub-agents via Claude Code's `Task()` dispatch, which has no in-skill equivalent in Codex. They remain a Claude Code plugin.

For deterministic checks without Claude Code (manifest-vs-disk consistency, frontmatter validity, hook event-name case) on **any** tool's artifacts, use the standalone Python validator:

```bash
curl -fsSL -o /usr/local/bin/nlpm-check \
  https://raw.githubusercontent.com/xiaolai/nlpm/main/bin/nlpm-check
chmod +x /usr/local/bin/nlpm-check
nlpm-check .
```

## Skills in this collection

| Skill | What it teaches |
|-------|-----------------|
| `$nlpm-rules` | The 50 Rules of Natural Language Programming (R01–R51) |
| `$nlpm-scoring` | 100-point penalty rubric per artifact type |
| `$nlpm-conventions` | Universal floor: SKILL.md open spec, AGENTS.md, naming |
| `$nlpm-conventions-claude` | Claude Code overlay: `.claude/` paths, plugin.json, hook events |
| `$nlpm-conventions-codex` | Codex CLI overlay: `.codex/config.toml`, `.codex-plugin/`, `.agents/skills/` |
| `$nlpm-conventions-antigravity` | Antigravity + Gemini CLI overlay |
| `$nlpm-patterns` | NL programming patterns + anti-patterns |
| `$nlpm-vocabulary` | Controlled-vocabulary discipline (R51) |
| `$nlpm-testing` | NL-TDD spec format |
| `$nlpm-security` | Security pattern database for executable artifacts |
| `$nlpm-orchestration` | Multi-agent workflow patterns (Claude-lineage; informational on Codex) |
| `$nlpm-writing-skills` … `$nlpm-writing-prompts` | Authoring guides per artifact type |

Source and full Claude Code plugin: <https://github.com/xiaolai/nlpm>

---
> Source: [xiaolai/nlpm](https://github.com/xiaolai/nlpm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
