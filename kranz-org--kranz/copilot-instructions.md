## kranz

> - Start every task from an up-to-date `main` and create a dedicated branch

# Repository workflow for coding agents

- Start every task from an up-to-date `main` and create a dedicated branch
  before changing files. This applies to every change, including small fixes,
  documentation, CI, and release preparation.
- Never commit or push task work directly to `main`.
- Push the task branch, open a pull request, wait for required CI, and merge the
  pull request. Prefer squash merge unless preserving individual commits is
  important to the change.
- Do not rewrite published history.
- Develop a minor release through task-sized branches merged into `main`. Use a
  `release/vX.Y.Z` branch only for final release preparation and stabilization,
  not as a long-lived replacement for `main`.

---
> Source: [kranz-org/kranz](https://github.com/kranz-org/kranz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
