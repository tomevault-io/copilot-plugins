## patina

> - `src/` contains the CLI, deterministic analysis, rewrite verification, and backends.

# Patina repository map

- `src/` contains the CLI, deterministic analysis, rewrite verification, and backends.
- `api/` and `playground/` contain the hosted API and browser UI.
- `SKILL.md`, `core/`, `patterns/`, `document-types/`, and `personas/` contain product assets.
- `tests/` contains unit, CLI, integration, and browser fixtures.
- `scripts/` contains development, benchmark, and release tooling.

Development uses `main` and short-lived feature branches. Useful references:

- [Contributor guide](CONTRIBUTING.md)
- [Git and release commands](docs/WORKFLOW.md)
- [Current architecture](docs/ARCHITECTURE.md)
- [Test commands and coverage](docs/QA.md)

`npm test` runs unit and CLI fixtures. `npm run lint` runs static checks.
`npm run test:browser` runs Chromium fixtures. Other commands are in `package.json`.

---
> Source: [devswha/patina](https://github.com/devswha/patina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
