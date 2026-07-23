## linq2db

> Entry point for **Claude Code** working on linq2db. The canonical, agent-agnostic contributor rules live in `AGENTS.md` — this file imports them, then layers Claude-Code-specific mechanics on top.

# CLAUDE.md

Entry point for **Claude Code** working on linq2db. The canonical, agent-agnostic contributor rules live in `AGENTS.md` — this file imports them, then layers Claude-Code-specific mechanics on top.

@AGENTS.md

## Claude Code specifics

- `.claude/` is a **symlink to `.agents/`**. Skills, subagents, hooks, scripts, docs, and the knowledge base all live under `.agents/` and are discovered through the symlink — so `.claude/...` and `.agents/...` paths both resolve. Layout, settings precedence, and skill discovery: [.agents/docs/claude-setup.md](.agents/docs/claude-setup.md).
- The operational overlay imported below is Claude-Code-specific (shell/tool rules, permission-friendly Bash patterns, dedicated-tools-over-CLI, worktree mechanics, `.claude/` curation carry-over, subagent verification, skill-based workflows). It complements — never overrides — the rules in `AGENTS.md`.

@.agents/docs/agent-rules.md

---
> Source: [linq2db/linq2db](https://github.com/linq2db/linq2db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
