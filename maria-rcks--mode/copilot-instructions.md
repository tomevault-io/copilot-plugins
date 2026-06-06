## mode

> - Edit product code in `.mode/t3code`.

# Agent Notes

- Edit product code in `.mode/t3code`.
- After edits there, run `bun mode export` from this repo.
- Commit Mode-owned files only: `patches/**`, `assets/**`, `scripts/**`, docs, config, and lock files.
- Verify product changes in `.mode/t3code` with `bun fmt`, `bun lint`, and `bun typecheck`.
- Do not run `bun test`; use `bun run test`.
- Keep commits atomic and use Conventional Commit style, like `fix(area): description`.
- Add deterministic tests for behavior changes; avoid external network access in tests.

---
> Source: [maria-rcks/mode](https://github.com/maria-rcks/mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
