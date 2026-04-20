## clawwork

> Canonical rules are maintained in `CLAUDE.md` and `.claude/rules/`.

# ClawWork Copilot Instructions

Canonical rules are maintained in `CLAUDE.md` and `.claude/rules/`.
Read those files for the complete rule set.

Key entry points:

- `CLAUDE.md` — project overview, commands, verification gate
- `.claude/rules/architecture.md` — product identity, layer ownership, invariants
- `.claude/rules/frontend.md` — TypeScript, React, styling
- `.claude/rules/main-process.md` — Electron main process, IPC, WebSocket
- `.claude/rules/message-persistence.md` — message write paths (CRITICAL)
- `.claude/rules/git-conventions.md` — commit format, PR budget

Run `pnpm check` before claiming completion.

---
> Source: [clawwork-ai/ClawWork](https://github.com/clawwork-ai/ClawWork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
