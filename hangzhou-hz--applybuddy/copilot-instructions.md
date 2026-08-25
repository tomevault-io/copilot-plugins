## applybuddy

> This repository is the complete ApplyBuddy workspace. Keep `config/`, `resumes/`, `skills/`, `references/`, and `tracking/` together, and resolve all paths from the repository root. The folders under `skills/` are the canonical Skill sources; neither workflow is a standalone package.

# ApplyBuddy Workspace Instructions

This repository is the complete ApplyBuddy workspace. Keep `config/`, `resumes/`, `skills/`, `references/`, and `tracking/` together, and resolve all paths from the repository root. The folders under `skills/` are the canonical Skill sources; neither workflow is a standalone package.

## Route requests

- When the user says `Set up ApplyBuddy for me.`, `帮我初始化 ApplyBuddy。`, or otherwise requests setup, read `skills/applybuddy/SKILL.md` and `references/setup-workflow.md` before acting.
- When setup explicitly mentions Simplify, also read `skills/applybuddy-simplify/SKILL.md` and `skills/applybuddy-simplify/references/simplify-codex-playbook.md` for Simplify readiness only. Do not create a second User Layer.
- For the standard workflow, read `skills/applybuddy/SKILL.md`.
- For the Simplify-assisted workflow, read `skills/applybuddy-simplify/SKILL.md`.

## Shared source of truth

Both workflows use the same live files:

- `config/profile.yml`
- `config/preferences.yml`
- `config/answers.yml`
- approved resume files in `resumes/`
- live operational CSVs in `tracking/`

Example files are schemas, never candidate facts. Do not overwrite an existing live file during setup. Initialize missing tracking files internally from their matching `*.example.csv` schemas; normal users should not have to edit tracking CSVs manually.

## Safety

Never invent consequential candidate facts. Preserve `ask_user` for unknown immigration, sponsorship, compensation, demographic, disability, criminal, education, employment, conflict, legal, or employer-specific information. Stop for CAPTCHA, login/OTP, user-controlled permissions, unsupported attestations, and other handoff conditions defined in `references/safety-and-boundaries.md`.

Do not publish live configuration, resumes, tracking data, browser state, credentials, or personal documents.

---
> Source: [hangzhou-hz/ApplyBuddy](https://github.com/hangzhou-hz/ApplyBuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
