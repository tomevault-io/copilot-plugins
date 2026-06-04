## ai-rules-sync-906

> agentsync converts between AI coding-agent rule files (AGENTS.md, CLAUDE.md, .cursorrules, Copilot, Windsurf) and can scaffold a fresh AGENTS.md. The same engine in src/core/agentsync.js powers both the CLI and the browser playground.

# agentsync

agentsync converts between AI coding-agent rule files (AGENTS.md, CLAUDE.md, .cursorrules, Copilot, Windsurf) and can scaffold a fresh AGENTS.md. The same engine in src/core/agentsync.js powers both the CLI and the browser playground.

## Build & test commands

- Install: `no install needed (Node 18+, zero dependencies)`
- Dev server: `npm run web   # serves the playground at http://localhost:5173`
- Test: `npm test`

Always run the test and lint commands before finishing a task.

## Architecture

- `src/core/agentsync.js` is the engine and the single source of truth. It imports nothing.
- `bin/agentsync.mjs` is the CLI; `web/` is the browser playground. Both import the engine directly — keep it dependency-free and isomorphic (must run in Node and the browser).
- Adding a tool format = add an entry to `FORMATS`, a branch in `parse()`, and a renderer in `RENDERERS`. Nothing else should need to change.
- Things that don't translate between tools (Cursor `globs`, Claude `@imports`) must surface as `warnings`, never be silently dropped.

## Code style & conventions

- Match the style of surrounding code; do not reformat unrelated lines.
- Prefer small, focused changes and clear names over cleverness.
- Write idiomatic JavaScript (ESM, zero-dependency). Do not add runtime dependencies.
- Add or update a case in `test/run.mjs` for any behavior change.

## Before opening a PR

- [ ] Tests and lint pass locally.
- [ ] No unrelated files changed.
- [ ] Public APIs and docs updated if behavior changed.

---
> Source: [VergeWarlord/ai-rules-sync-906](https://github.com/VergeWarlord/ai-rules-sync-906) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
