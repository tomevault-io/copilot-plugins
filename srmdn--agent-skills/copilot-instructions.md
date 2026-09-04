## agent-skills

> This repository contains reusable agent skills maintained by srmdn. Keep each

# AGENTS.md

## Scope

This repository contains reusable agent skills maintained by srmdn. Keep each
skill self-contained, narrowly scoped, and safe to run in a user's project.

## Repository rules

- Read the target skill and relevant repository files before editing.
- Keep generated project checkpoints such as `HANDOFF.md` out of this repo.
- Never include credentials, tokens, private paths, or conversation transcripts.
- Keep changes focused and update the skill's documentation and evaluation notes
  when its contract changes.
- Validate changed skills with the `skill-creator` validation script.
- Run `git diff --check` before committing.
- Use standard Markdown and avoid em dash characters in documentation.

## Skill contract

Every published skill must have a `SKILL.md` with valid YAML frontmatter,
clear activation guidance, bounded scope, safety rules, an actionable
workflow, and an explicit stopping condition. Add supporting files only when
they provide concrete value.

---
> Source: [srmdn/agent-skills](https://github.com/srmdn/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
