## katana-ast-lint

> - This repository provides a thin CLI (`kal`) in addition to the library API. Rule implementation and configuration interpretation must remain in the library.

# Project Rules

- This repository provides a thin CLI (`kal`) in addition to the library API. Rule implementation and configuration interpretation must remain in the library.
- `README.md`, `CHANGELOG.md`, and public Markdown under `docs/` must be written in English.
- The repository default branch is `master`. Documentation-only, OpenSpec-only, and workflow-maintenance changes that do not affect the library body should be done directly on `master`.
- CI/CD and release dispatch settings must target `master`; do not leave `main` as a default or release reference.
- Use `scripts/openspec` instead of calling a bare `openspec` command from agent instructions or local workflow docs.
- Run `just check` before commit and release handoff.
- Do not bypass `lefthook`, Clippy, AST lint, or OpenSpec validation.
- Before clearing local diffs or stashes, inspect their contents and decide whether each item should be incorporated, ignored, or kept for user review.

---
> Source: [HiroyukiFuruno/katana-ast-lint](https://github.com/HiroyukiFuruno/katana-ast-lint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
