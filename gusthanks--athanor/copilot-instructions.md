## athanor

> This repository contains Athanor, an open-source project operating system packaged

# AGENTS.md

## Purpose

This repository contains Athanor, an open-source project operating system packaged
as a Codex plugin with multiple cooperating skills.

## Sources of truth

- Product overview: `README.md`
- User guide: `docs/HANDBOOK.md`
- Architecture: `docs/ARCHITECTURE.md`
- Plugin manifest: `plugins/athanor/.codex-plugin/plugin.json`
- Runtime instructions: `plugins/athanor/skills/*/SKILL.md`

## Working rules

- Use `YYYY-MM-DD` in documentation, examples, interfaces, and generated records.
- Keep machine-facing names clear even when user-facing language is alchemical.
- Never include private knowledge, internal brand material, credentials, or
  identifying examples from another project in this public repository.
- Treat inspection as read-only unless a user explicitly authorizes a mutation.
- Never overwrite existing project context while bootstrapping Athanor.
- Keep organization dry-run by default, plan-first, collision-safe, and auditable.
- Never print secret values or read secret-file contents for classification.
- Prefer Python standard-library implementations for portable local utilities.

## Verification

Run:

```powershell
python -m unittest discover -s tests -v
python plugins/athanor/scripts/validate_all.py
```

Do not publish or push externally without explicit authorization.

---
> Source: [gusthanks/athanor](https://github.com/gusthanks/athanor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
