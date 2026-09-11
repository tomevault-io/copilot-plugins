## dst-character-studio

> This is an art authoring/validation tool, not an automatic gameplay mod generator.

# Agent guide

This is an art authoring/validation tool, not an automatic gameplay mod generator.

- Read git status before edits. Keep changes in this repository.
- Use `python -m dst_studio` from the repo root. Public examples are source
  references; initialize an ignored workspace project before editing them.
- Never distribute original game data, Klei executables, user machine config,
  credentials, logs or third-party workshop assets.
- Tool code is MIT; `examples/lucia` and `docs/media` have separate artwork terms.
- Preserve shared drawing coordinates and out-of-canvas pivots. Frozen binding
  profiles cannot be inferred from layer order or silently changed during art edits.
- Stage checks are diagnostic. Do not claim local replays are in-game evidence.
- Run unit tests, ink tests and distribution scan. Compiler changes also need a
  local Wilson roundtrip and a rebuilt example. No game launch or deployment is
  part of the automated tool workflow.
- Keep one project per server process. Do not switch a collaborator's project or
  overwrite their newer revision. A 409 conflict requires download/merge/reload.
- Do not publish, push, add a remote or change firewall rules unless requested.

---
> Source: [0xleonvibe/dst-character-studio](https://github.com/0xleonvibe/dst-character-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
