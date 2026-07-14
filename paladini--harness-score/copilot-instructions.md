## core

> Non-negotiable conventions for the harness-score monorepo


- The scanner is deterministic by contract: no model calls, no network, no
  time-dependent output. If a proposed check needs judgment, it does not
  belong in the CLI — it belongs in the guide as advice.
- `packages/cli` has zero runtime dependencies. Do not add one.
- Rubric changes touch three files together: `packages/cli/src/score.ts`,
  `docs/guide/maturity-model.md`, `docs/guide/measure-and-improve.md`.
- Check IDs are stable public API (they are documentation anchors and appear
  in users' CI logs). Never renumber, reuse, or delete an ID — deprecate by
  making the check always-pass if truly needed.
- This repository must keep scoring L4 on `npm run scan`.

---
> Source: [paladini/harness-score](https://github.com/paladini/harness-score) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
