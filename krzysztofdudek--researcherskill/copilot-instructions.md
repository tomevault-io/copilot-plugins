## researcherskill

> This repository exists solely so the author can develop and version `researcher.md`. That file is the complete, standalone researcher skill — people copy it into their own setups. Nothing in this repo (CLAUDE.md, CHANGELOG.md, README.md, CI, etc.) may affect the skill's mechanics. All behavior must be self-contained in `researcher.md`.

# ResearcherSkill

## Purpose

This repository exists solely so the author can develop and version `researcher.md`. That file is the complete, standalone researcher skill — people copy it into their own setups. Nothing in this repo (CLAUDE.md, CHANGELOG.md, README.md, CI, etc.) may affect the skill's mechanics. All behavior must be self-contained in `researcher.md`.

## Sync rule

`researcher.md` and `~/.claude/skills/researcher/SKILL.md` are clones. Any change to one MUST be applied to the other in the same operation. Never leave them out of sync.

## Versioning

This project uses [Semantic Versioning](https://semver.org/) and maintains a [CHANGELOG.md](CHANGELOG.md) following the [Keep a Changelog](https://keepachangelog.com/) format.

When the user says "bump version":
1. Move `[Unreleased]` entries in `CHANGELOG.md` into a new version section with today's date
2. Update the comparison links at the bottom of `CHANGELOG.md`
3. Commit and tag with `v{version}`

Tags are created via the CI pipeline automatically — do not create tags manually.

---
> Source: [krzysztofdudek/ResearcherSkill](https://github.com/krzysztofdudek/ResearcherSkill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
