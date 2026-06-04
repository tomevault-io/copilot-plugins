## codex-ui-board-skills

> Guidance for Codex and other AI agents working in this repository.

# AGENTS.md

Guidance for Codex and other AI agents working in this repository.

## Project Overview

Codex UI Board Skills is an open-source collection of reusable Codex skills for UI design-system starter kits. Each skill should provide:

- an installable `codex-skill` bundle
- a standalone `preview.html`
- CSS implementation references
- design tokens
- a Tailwind preset
- screenshots
- clear documentation

The repository is focused on dual-theme UI systems, design tokens, previews, and developer experience for SaaS dashboards, admin panels, AI tools, internal tools, and productivity apps.

## Repository Conventions

- Do not break the existing top-level skill folder structure.
- Keep examples simple, standalone, and inspectable.
- Prefer small focused commits.
- Keep documentation in professional English.
- Keep generated or platform-specific clutter out of the repository.
- Keep `preview.html`, CSS, tokens, Tailwind preset, and `codex-skill/references/` aligned.

## Adding a New Skill

Use this structure:

```text
SkillName/
├── README.md
├── package.json
├── preview.html
├── index.css
├── SkillName.css
├── SkillName.tokens.json
├── tailwind.preset.cjs
├── screenshots/
└── codex-skill/
    ├── SKILL.md
    ├── agents/
    ├── assets/
    └── references/
```

The skill name in `codex-skill/SKILL.md` should be kebab-case. The top-level folder should be PascalCase.

## Updating Previews

- Keep previews standalone.
- Use local CSS and assets.
- Preserve light and dark theme support when a skill is dual-theme.
- Update screenshots when visual output changes.
- Keep copy realistic and concise.
- Avoid adding external scripts unless there is a strong reason and it is documented.

## Coding and Style Rules

- Prefer readable CSS and semantic token names.
- Use design tokens instead of scattering hardcoded values.
- Keep JavaScript in previews small and easy to audit.
- Avoid unrelated refactors.
- Avoid adding build requirements unless the project explicitly adopts them.

## Documentation Expectations

When changing a skill, update the relevant README and reference documentation. Include:

- what the skill is for
- how to preview it
- how to install it
- how to use it in Codex prompts
- how to use CSS or Tailwind assets directly

## Guardrails

- Do not remove existing skills without a clear reason.
- Do not rename folders casually; installation instructions depend on stable paths.
- Do not invent fake badges, metrics, contributors, downloads, or production users.
- Do not claim CI or npm publishing exists unless it has been added and verified.

---
> Source: [Dezoff-max/codex-ui-board-skills](https://github.com/Dezoff-max/codex-ui-board-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
