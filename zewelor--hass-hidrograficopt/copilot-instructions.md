## hass-hidrograficopt

> This repository uses a shared AI agent instruction system. **All instructions are in [`AGENTS.md`](AGENTS.md).**

# Gemini Instructions

This repository uses a shared AI agent instruction system. **All instructions are in [`AGENTS.md`](AGENTS.md).**

Read `AGENTS.md` completely before starting any work. It contains:

- Project overview and integration identifiers
- Package structure and architectural rules
- Code style, validation commands, and quality expectations
- Home Assistant patterns (config flow, coordinator, entities, services)
- Error recovery strategy and breaking change policy
- Workflow rules (scope management, translations, documentation)

## Quick Reference

- **Domain:** `hidrograficopt`
- **Title:** Instituto Hidrográfico Integration
- **Class prefix:** `InstitutoHidrogrfico`
- **Main code:** `custom_components/hidrograficopt/`
- **Validate:** `script/check` (type-check + lint + spell)
- **Test:** `script/test`
- **Run HA:** `./script/develop`

## Path-Specific Instructions

Additional domain-specific guidance is available in `.github/instructions/*.instructions.md`.
These files use `applyTo` globs to indicate which files they cover.
Consult the relevant instruction file when working on specific file types:

- `python.instructions.md` — Python style, async patterns, HA imports
- `entities.instructions.md` — Entity platform patterns, inheritance
- `config_flow.instructions.md` — Config flow, reauth, discovery
- `coordinator.instructions.md` — DataUpdateCoordinator patterns
- `api.instructions.md` — API client, exception hierarchy
- `services_yaml.instructions.md` — Service action definitions
- `translations.instructions.md` — Translation file structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zewelor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zewelor)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
