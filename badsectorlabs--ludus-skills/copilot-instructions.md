## ludus-skills

> This repository is a skills collection, not an application runtime. Keep changes scoped to `skills/`.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a skills collection, not an application runtime. Keep changes scoped to `skills/`.

- `skills/<skill-name>/SKILL.md`: core agent instructions for one skill.
- `skills/<skill-name>/references/*.md`: detailed source-backed reference material.
- `skills/<skill-name>/agents/openai.yaml`: UI metadata (`display_name`, `short_description`, `default_prompt`).
- Root docs: `README.md` and this guide.

Current skills are `environment-guide`, `ludus-cli`, `range-config`, and `troubleshooting`.

## Build, Test, and Development Commands
There is no compile/build step in this repo. Validate by installing skills locally:

```bash
npx skills add .
npx skills list
```

Use the published install path when testing consumer behavior:

```bash
npx skills add badsectorlabs/ludus-skills
```

## Coding Style & Naming Conventions
- Use Markdown with concise, actionable instructions.
- Skill folders must be kebab-case (example: `range-config`).
- Keep `SKILL.md` frontmatter minimal: `name` and `description`.
- Put detailed docs in `references/`; keep `SKILL.md` focused on workflow and decision guidance.
- For long reference files, include a `## Table of Contents`.

## Testing Guidelines
- Prefer factual updates sourced from official Ludus documentation.
- After edits, run local install checks (`npx skills add ...`) and confirm skills are discoverable.
- Validate that links/paths in `SKILL.md` resolve correctly (for example, `references/schema.md`).
- If behavior changed, test one realistic prompt per modified skill.

## Commit & Pull Request Guidelines
- Follow conventional-style commit prefixes with emoji, as used in repo history:
  - `feat: ✨ ...`
  - `docs: 📚 ...`
- Keep subject lines imperative and concise.
- Commit only files related to your change.
- Include DCO sign-off on commits (`git commit -s`).
- PR/MR descriptions should state:
  - what changed,
  - why it changed,
  - which skills/files were updated,
  - how you validated locally.

---
> Source: [badsectorlabs/ludus-skills](https://github.com/badsectorlabs/ludus-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
