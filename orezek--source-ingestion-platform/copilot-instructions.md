## source-ingestion-platform

> Gemini agents working in this repository must follow the repository-wide instructions in `AGENTS.md`.

# Gemini Agent Instructions

Gemini agents working in this repository must follow the repository-wide instructions in `AGENTS.md`.

## Mandatory Rule Sources

1. `AGENTS.md` (primary repository instructions)
2. `.aiassistant/rules/monorepo.md` (monorepo engineering rules referenced by `AGENTS.md`)

## Precedence

- System and tool safety instructions take precedence.
- Then follow `AGENTS.md` and `.aiassistant/rules/monorepo.md`.
- If documentation conflicts with repository reality, prefer the real configuration (`package.json`, `turbo.json`, `pnpm-workspace.yaml`) and update docs accordingly.

## Scope

- Applies to the entire repository.
- If a deeper `AGENTS.md` exists in a subdirectory, follow it in addition to root rules.

Do not treat this file as an alternative rule set. Use it as an entry point that delegates to `AGENTS.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orezek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/orezek)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
