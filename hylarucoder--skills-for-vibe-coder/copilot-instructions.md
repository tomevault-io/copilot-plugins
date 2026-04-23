## skills-for-vibe-coder

> This repo is a collection of skills. The core content lives under `skills/`, with one directory per skill (kebab-case names like `skills/ast-grep-rule-crafter`). Each skill directory contains a `SKILL.md` entrypoint and may include `references/` with supporting documents (for example, `skills/doc-consistency-reviewer/references/checklist.md`). Root docs include `README.md` and `README_CN.md`.

# Repository Guidelines

## Project Structure & Module Organization

This repo is a collection of skills. The core content lives under `skills/`, with one directory per skill (kebab-case names like `skills/ast-grep-rule-crafter`). Each skill directory contains a `SKILL.md` entrypoint and may include `references/` with supporting documents (for example, `skills/doc-consistency-reviewer/references/checklist.md`). Root docs include `README.md` and `README_CN.md`.

## Build, Test, and Development Commands

There are no build or test scripts in this repository. The main operational command is the install copy:

```bash
cp -r skills/* ~/.claude/skills/
```

Use it to install or refresh the skills locally. Validate changes by reading the affected Markdown files and checking internal links.

## Coding Style & Naming Conventions

Content is Markdown-first. Keep paragraphs short, use headings for structure, and fence command examples. Match existing formatting (tables in `README.md`, ordered headings in `SKILL.md` files). Naming conventions:

- Skill directories: kebab-case (e.g., `clean-code-reviewer`).
- Entrypoint file: `SKILL.md` (uppercase, consistent across skills).
- Reference docs: short, descriptive names under `references/` (e.g., `rule-syntax.md`).

## Testing Guidelines

No automated tests are defined. For changes, do a quick manual pass:

- Open the edited `SKILL.md` and any `references/` files.
- Verify links and code blocks render as expected.
- Ensure examples match the skill’s stated workflow.

## Commit & Pull Request Guidelines

Commit messages follow a conventional format: `type(scope): summary`. Examples from history include `docs: ...` and `refactor(skills): ...`. Keep summaries short and scoped to the skill or docs you touched.

For pull requests, include a clear description of what changed, list affected skills, and mention any updates to `README.md`/`README_CN.md` when you add or rename skills.

## Agent-Specific Notes

Skills are the source of truth for behavior. If you change a skill, update its `SKILL.md` first and keep any referenced files in `references/` in sync.

---
> Source: [hylarucoder/skills-for-vibe-coder](https://github.com/hylarucoder/skills-for-vibe-coder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
