## cairn

> Plugin marketplace for making codebases ready for autonomous AI development.

# Cairn

Plugin marketplace for making codebases ready for autonomous AI development.

## Repository layout

- `plugins/` — self-contained plugins (`skills/<name>/` with nested `references/`, plus `hooks/`, `scripts/`, manifests)
- `.claude-plugin/marketplace.json` — Claude Code marketplace registry
- `.agents/plugins/marketplace.json` — Codex marketplace registry
- `schemas/` — JSON schemas for manifests and SKILL frontmatter
- `tools/` — validators and markdownlint custom rules
- `docs/DESIGN_GUIDELINES.md` — plugin authoring rules
- `docs/DEVELOPMENT_GUIDE.md` — local development and CI

## Before changing plugins

Run `mise run build` (lint + reference integrity). See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Size limits

- `SKILL.md`: max 400 lines
- `references/*.md`: max 150 lines each
- Split content across reference files when limits are exceeded

---
> Source: [krokoko/cairn](https://github.com/krokoko/cairn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
