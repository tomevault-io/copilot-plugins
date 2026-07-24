## shaft-engine

> - Imported `AGENTS.md` is canonical (including binding `act-as-fable`);

# CLAUDE.md

@AGENTS.md

## Claude Adapter

- Imported `AGENTS.md` is canonical (including binding `act-as-fable`);
  do not restate it or append logs.
- Load one matching `.agents/skills/` bridge only when its trigger applies;
  native Graphify via `.claude/skills/graphify`.
- Keep plans and final responses proportional to the task; stop when the
  requested behavior is verified.

---
> Source: [ShaftHQ/SHAFT_ENGINE](https://github.com/ShaftHQ/SHAFT_ENGINE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
