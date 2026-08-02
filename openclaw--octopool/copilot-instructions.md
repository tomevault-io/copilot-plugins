## octopool

> This is a deepsec scanning workspace. Each registered project keeps

# Agent setup

This is a deepsec scanning workspace. Each registered project keeps
hand-curated context in `data/<id>/INFO.md` — open the relevant one
when asked to scan or refresh project setup.

## Common tasks

- **Set up a project for scanning**: read
  `node_modules/deepsec/SKILL.md`, then fill `data/<id>/INFO.md` from
  the target codebase.
- **Add a new project**: run `deepsec init-project <root>` — it
  scaffolds `data/<id>/` and prints/writes the setup prompt for the
  new project.
- **Write a custom matcher** (only after a real true-positive shows you
  a pattern worth keeping): read
  `node_modules/deepsec/dist/docs/writing-matchers.md`.

## Reference

The deepsec skill is at `node_modules/deepsec/SKILL.md` (after
`pnpm install`). The full docs ship at
`node_modules/deepsec/dist/docs/`.

---
> Source: [openclaw/octopool](https://github.com/openclaw/octopool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
