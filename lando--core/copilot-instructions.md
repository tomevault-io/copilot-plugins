## core

> - Each `README.md` is an executable Leia spec, not just documentation. **Do not run locally** — they are CI-only and can mutate host state.

# AGENTS.md

- Each `README.md` is an executable Leia spec, not just documentation. **Do not run locally** — they are CI-only and can mutate host state.
- Leia expects specific section headings: `Start up tests`, `Verification commands`, `Destroy tests`. `npm run test:leia` targets `Destroy tests` for cleanup.
- When changing example behavior, keep the `README.md` commands in the same directory in sync.
- When adding, removing, or renaming specs, update the corresponding jobs in `.github/workflows/pr-core-tests.yml` and the `pr-setup-{linux,windows,macos}-tests.yml` workflows.
- `examples/.lando.yml` loads `@lando/core` from `..` — examples exercise this checkout, not an installed release.

---
> Source: [lando/core](https://github.com/lando/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
