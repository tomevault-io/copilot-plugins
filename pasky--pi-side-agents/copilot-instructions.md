## pi-side-agents

> - Package: `pi-side-agents` (Pi extension + skill for parallel child-agent orchestration).

# AGENTS quick context

- Package: `pi-side-agents` (Pi extension + skill for parallel child-agent orchestration).
- Core code: `extensions/side-agents.ts`; setup skill: `skills/agent-setup/SKILL.md`.
- Public contract to preserve: `/agent`, `/agents`, tools `agent-start`, `agent-check`, `agent-wait-any`, `agent-send`.
- Runtime assumptions: `tmux` + git worktrees; shared registry at `.pi/side-agents/registry.json`.
- Validation: `npm run test:unit` and (real-runtime) `npm run test:integration`.
- Always commit completed work before reporting the task done.

---
> Source: [pasky/pi-side-agents](https://github.com/pasky/pi-side-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
