## wateringsystem

> The source of truth for project rules is the root `CLAUDE.md` (plus `firmware/CLAUDE.md` for firmware specifics). Read those first. Key rules:

# Copilot Instructions — WateringSystem

The source of truth for project rules is the root `CLAUDE.md` (plus `firmware/CLAUDE.md` for firmware specifics). Read those first. Key rules:

## Language

- English everywhere: code, comments, commit messages, PR titles/descriptions, issues, documentation. The repository is public.

## Frozen legacy code

- `src/`, `include/`, `data/`, `test/`, and `platformio.ini` are **FROZEN** — the production greenhouse unit runs this code.
- Never modify these paths on `main` or feature branches. Legacy patches go only through the `arduino-maintenance` branch.

## Active development

- All new work happens in `firmware/` — ESP-IDF v6.0.1, C++.
- Interface-based architecture (`ISensor`, `IActuator`, `IModbusClient`, ...) with RAII; prefer `std::string` over raw buffers.
- Include guards in all headers: `WATERINGSYSTEM_PATH_FILE_H` (e.g. `WATERINGSYSTEM_SENSORS_SOILSENSOR_H`).

## Git

- Conventional commits, written in English.
- Never squash-merge — always use merge commits (squash breaks spec-kit branch chaining).

See `docs/PRD-esp-idf-migration.md` for project scope and `docs/prd/` for the PR breakdown.

---
> Source: [cryptotomte/WateringSystem](https://github.com/cryptotomte/WateringSystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
