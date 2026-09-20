## sno-station

> Instructions for any coding agent working in this repository.

# AGENTS.md

Instructions for any coding agent working in this repository.

- **What this is.** Sno Station: shared memory, Reach messaging, squad skills and a nightly
  improvement loop for Claude Code, Codex and OpenClaw. Read `llms.txt` for the map.
- **Fixes land here first.** This is the source of truth for the product code. Do not point users
  or docs at any other repository.
- **Checks before you hand back work.** `npm run build --workspaces && npm run typecheck && npm run lint`,
  plus the unit tests of the package you touched. An empty test run is a failure, not a pass.
- **Sign off.** Every commit carries `Signed-off-by` (DCO). No CLA.
- **Never add** benchmark numbers or vendor comparisons to docs; numbers live in `evals/` with a
  receipt. Never name one model vendor as the careful one and the other as the fast one.
- **Never commit** credentials, private host names, absolute home paths, or links to private
  documents. The CI scan rejects them.
- **Public words.** The category word is `squad` (lower-case). The product is Sno Station. Package
  names are internal and do not appear on the README first screen.

---
> Source: [sno-ai/sno-station](https://github.com/sno-ai/sno-station) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
