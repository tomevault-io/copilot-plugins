## adr-writing

> ADR writing guidelines (clarity, flow, examples)


- Prefer a coherent narrative: **context → problem → constraints → decision → consequences → examples**
- Keep responsibility splits explicit (e.g. adapter lowering vs codec encode/decode vs driver transport/normalization)
- If an ADR discusses capabilities, follow `.cursor/rules/capabilities-ownership.mdc` for wording
- If you need to describe sequencing (order of operations), keep it **short** and place it near the responsibilities/decision; avoid long procedural sections that read out-of-place
- Use fenced code blocks for SQL (` ```sql `) and TS (` ```ts `); avoid nested backticks inside inline code
- Worked examples should be end-to-end and make it obvious which layer owns each step (lane, adapter, codec, driver, runtime)

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
