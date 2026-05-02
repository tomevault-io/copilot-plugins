## codex-bmad-skills

> This repository provides BMAD skills natively for OpenAI Codex.

# BMAD for OpenAI Codex

This repository provides BMAD skills natively for OpenAI Codex.

## Runtime Contract

- Target runtime: OpenAI Codex.
- Skill install roots:
  - Global: `~/.agents/skills`
  - Project: `<project>/.agents/skills`
- State/config format: YAML files edited by `yq` (v4+).

## Skill Sources in This Repository

- Canonical source during migration: `skills/*`

## Trigger Intents

Use these intent IDs in prompts:

- `bmad:init`, `bmad:status`, `bmad:next`
- `bmad:product-brief`, `bmad:research`, `bmad:brainstorm`
- `bmad:prd`, `bmad:tech-spec`, `bmad:prioritize`
- `bmad:architecture`, `bmad:gate-check`
- `bmad:sprint-plan`, `bmad:create-story`
- `bmad:dev-story`, `bmad:code-review`
- `bmad:ux-design`, `bmad:user-flow`
- `bmad:ideate`, `bmad:research-deep`
- `bmad:create-skill`, `bmad:create-workflow`

## Migration Safety Rules

- Do not use deprecated custom prompts as a compatibility layer.
- Prefer project-local skills over global when names collide.
- Keep workflow state explicit in YAML files under `<project>/bmad/`.
- Enforce skills path isolation: any script, template, or referenced resource used by `skills/*` must remain within the `skills/` tree.
- Skills under `skills/*` must not read, import, execute, or reference paths outside `skills/` (for example `../installers`, `../docs`, or any parent path escape).

## Language Settings

When `bmad/project.yaml` is present, apply:

- `language.communication_language` for assistant communication.
- `language.document_output_language` for generated BMAD documents.

Fallback order:

1. `language.document_output_language`
2. `language.communication_language`
3. `"English"`

## Release and Commit Conventions

- Release tag format: `vMAJOR.MINOR.PATCH` (Semantic Versioning), optional pre-release suffix: `-rc.N` (for example: `v1.4.0-rc.1`).
- Release title format: `vMAJOR.MINOR.PATCH — <Title Case Summary>`.
- Accepted release title examples:
  - `v1.1.0 — Chat-Native Discovery Gate for Product Brief`
  - `v1.2.0 — Intent-Driven Onboarding & Skill Isolation`
- Before running release creation (`gh release create` / `git create release`), update `CHANGELOG.md` in `main` with all changes made since the previous release.
- Commit message format: Conventional Commits: `<type>(<scope>): <subject>`.
- Allowed commit types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `ci`, `build`, `perf`, `revert`.
- Keep the subject in imperative mood, lowercase start, and without a trailing period (for example: `feat(skills): add bmad status validator`).

---
> Source: [xmm/codex-bmad-skills](https://github.com/xmm/codex-bmad-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
