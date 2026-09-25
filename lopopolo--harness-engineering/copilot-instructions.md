## harness-engineering

> - Use the imported guide as the owning Python contract; do not infer a second

@AGENTS.md

# Claude Code

- Use the imported guide as the owning Python contract; do not infer a second
  style from nearby raw-source files.
- Prefer replacing defensive branches with one closed parser and a stronger
  domain object. Preserve useful error locations at the JSON boundary.
- Keep changes focused across models, decoders, tests, and executable adapters,
  then run the complete proof loop from the imported guide.
- Treat `uv` lockfiles as generated artifacts. Refresh them through `uv`; never
  edit them by hand.

---
> Source: [lopopolo/harness-engineering](https://github.com/lopopolo/harness-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
