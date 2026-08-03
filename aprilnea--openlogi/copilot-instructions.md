## openlogi

> - Path-scoped deep rules live in `.claude/rules/` and load automatically when you touch

@AGENTS.md

## Claude Code specifics

- Path-scoped deep rules live in `.claude/rules/` and load automatically when you touch
  matching files; the index is at the bottom of AGENTS.md.
- GPUI and gpui-component reference skills may be present locally under `.agents/skills/`
  (`gpui`, `gpui-component`, `gpui-entity`, `gpui-layout-and-style`, …; gitignored,
  per-developer). When they are available, consult them for GUI work instead of guessing
  gpui APIs from training data — the gpui pin moves and APIs drift.
- `crates/openlogi-gui/src/platform/CLAUDE.md` imports that directory's `AGENTS.md`
  (the macOS ObjC FFI contract) when you work in that subtree.

---
> Source: [AprilNEA/OpenLogi](https://github.com/AprilNEA/OpenLogi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
