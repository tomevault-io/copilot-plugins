## pact

> Read `CLAUDE.md` for the architecture and command reference.

# AGENTS.md -- Pact

Read `CLAUDE.md` for the architecture and command reference.

## Working Defaults

- Source `~/.profile` before project commands when feasible.
- Use Kindex at session start, before important decisions, and at session end.
- Default Pact runs are plan-only: Pact produces decomposition, contracts, and
  tests; the active Claude or Codex agent implements.
- Use `pact run <project> --implement` only when Pact should own implementation.
- If implementation exposes an infeasible contract, stop and reconcile the
  Pact artifacts instead of silently deviating.
- Use the repository skill at `skills/pact-engineer/SKILL.md` for substantial
  plan-first engineering work.
- Run focused tests while editing and the full suite before completion.
- Run architecture/done claims past Simulacrum and use `pact review` for the
  Advocate plus Simulacrum post-implementation gate.

---
> Source: [jmcentire/pact](https://github.com/jmcentire/pact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
