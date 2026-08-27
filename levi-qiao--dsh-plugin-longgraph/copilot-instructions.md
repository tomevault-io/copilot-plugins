## dsh-plugin-longgraph

> Community DeepSeek Harness plugin. Not affiliated with DeepSeek AI.

# AGENTS.md — dsh-plugin-longgraph

Community DeepSeek Harness plugin. Not affiliated with DeepSeek AI.

- Register skills on `ctx.skills`. Do not become a runtime node.
- DSH-only host copy. Timers are two independent `schedule_create` clocks.
- Peer range: `@deepseek-ai/cordis` `>=4.0.1 <5`, `@deepseek-ai/dsh-skill`
  `>=0.1.0-rc.5 <0.2.0`.
- After skill or plugin edits: `pnpm test`. After manifest edits, also
  confirm `dsh.bundle.patch` still points at `cordis.patch.yml`.

---
> Source: [levi-qiao/dsh-plugin-longgraph](https://github.com/levi-qiao/dsh-plugin-longgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
