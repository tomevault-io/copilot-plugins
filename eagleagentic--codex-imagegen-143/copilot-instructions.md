## codex-imagegen-143

> This repository packages a Codex skill for image generation through a fixed CLI version.

# Repository Guidelines

## Project Structure & Module Organization

This repository packages a Codex skill for image generation through a fixed CLI version.

- `skills/imagegen-0-143-0/SKILL.md` is the primary skill definition and workflow.
- `skills/imagegen-0-143-0/references/usage.md` contains command examples and operational notes.
- `skills/imagegen-0-143-0/agents/openai.yaml` defines the agent-facing display metadata.

Keep new skill-specific material under `skills/imagegen-0-143-0/`. Add reference material under `references/` only when it supports the main workflow without duplicating `SKILL.md`.

## Build, Test, and Development Commands

There is no build step or package manager configuration in this repository. Use lightweight validation commands:

```bash
rg --files
sed -n '1,220p' skills/imagegen-0-143-0/SKILL.md
sed -n '1,220p' skills/imagegen-0-143-0/references/usage.md
```

Use these to confirm file layout, review Markdown content, and catch accidental path changes. If you add tooling later, document the exact command here.

## Coding Style & Naming Conventions

Use Markdown for skill and reference documentation, and YAML for agent metadata. Keep headings descriptive, examples executable, and instructions direct. Prefer ASCII punctuation and fenced code blocks with `bash` for shell examples.

Skill directory names should be lowercase and versioned when behavior depends on a fixed dependency, for example `imagegen-0-143-0`. Keep prompt-sensitive instructions explicit: user prompts must remain verbatim, while execution details belong in separate instruction text.

## Testing Guidelines

No automated tests are currently configured. Validate documentation changes by reading the affected files and checking examples for correct paths, required flags, and absolute-path handling for image inputs.

For workflow changes, manually verify that examples still use:

- `npx --yes --package=@openai/codex@0.143.0`
- `codex exec --skip-git-repo-check`
- repeated `-i /absolute/path` arguments for input images
- a separate `Execution instruction:` line

## Commit & Pull Request Guidelines

The existing Git history uses concise, imperative commit messages, such as `Add image generation skill and usage documentation for Codex 0.143.0`. Follow that style.

Pull requests should include a short summary, the files changed, and the validation performed. For changes to command examples, mention whether they were manually reviewed or executed. Link related issues when available.

## Agent-Specific Instructions

Do not overwrite this file without checking the current root `AGENTS.md` first. Preserve the skill’s core constraint: the image prompt is passed verbatim, and only operational save-path or input-file mapping instructions may be added outside the prompt.

---
> Source: [eagleagentic/codex-imagegen-143](https://github.com/eagleagentic/codex-imagegen-143) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
