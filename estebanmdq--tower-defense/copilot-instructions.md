## tower-defense

> - **Commit incrementally while building a feature** - make a commit as each logical

# Project guidance

## Commits

- **Commit incrementally while building a feature** - make a commit as each logical
  unit of work completes (e.g. config, then the core system, then wiring, then
  tests), rather than batching everything into one commit at the end. This keeps the
  history reviewable and avoids tangled commits.
- Each commit should be coherent and self-contained, with a clear, concise message
  describing what changed and why.
- Run typecheck and tests before committing; don't commit a broken tree.
- When applying an OpenSpec change, you may group commits by stage; the archive of a
  change should land together with (or right after) its implementation, not bundled
  into the next feature's commit.
- Never include AI/assistant attribution in commit messages or code (matches the
  global rules).

## Project notes

- Stack: Phaser 3 + TypeScript (strict) + Vite. `npm run dev` / `npm test` /
  `npm run typecheck` / `npm run build`.
- Game design lives in `docs/GAME_DESIGN.md`; capability specs in `openspec/specs/`.
- Gameplay logic is kept framework-agnostic (no Phaser) so it is unit-testable;
  Phaser is used only in `src/scenes/` for rendering and input.

---
> Source: [EstebanMDQ/tower-defense](https://github.com/EstebanMDQ/tower-defense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
