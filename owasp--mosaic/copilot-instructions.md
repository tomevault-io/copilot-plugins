## autonomous-workflow

> End-to-end execution policy after plan approval


# MOSAIC Autonomous Workflow

Policy: see `AGENTS.md`. Verification: see `verifiable-goals.mdc`.

- For big changes, wait for human plan approval (`multi-agent-workflow.mdc` Phase 1) before editing.
- After approval, execute end-to-end unless blocked by auth/secrets, destructive actions, or material plan deviations.
- Prefer small, safe changes; avoid unrelated refactors.
- Use `npm run build:site` to verify Hugo output after substantive changes.
- Run long commands in background; monitor until completion.
- Do NOT commit or push unless explicitly asked.
- In commits do not add "made with cursor" lines.
- On substantive changes, recommend or run judge/subagent review before handoff.

---
> Source: [OWASP/MOSAIC](https://github.com/OWASP/MOSAIC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
