## aitofigma

> - Every task that creates, modifies, renames, or deletes code must invoke and complete the `good-readme` Skill workflow.

# Repository instructions

## Mandatory workflow for code changes

- Every task that creates, modifies, renames, or deletes code must invoke and complete the `good-readme` Skill workflow.
- Every task that creates, modifies, renames, or deletes code must invoke and complete the `gitwork` Skill workflow.
- These two Skill workflows are mandatory for code changes even when the user does not mention them explicitly. They are not required for read-only, research-only, or documentation-only tasks unless the user explicitly requests them. Follow each Skill's own applicability, safety, verification, and reporting requirements.

## Content boundaries

- Keep repository-level documentation, licensing, and contributor instructions at the project root.
- Keep the complete distributable Skill under `skills/ai-to-figma/`. Its `SKILL.md`, `agents/`, `assets/`, `references/`, runtime scripts, and runtime requirements must stay inside that directory.
- Treat `skills/ai-to-figma/` as the Skill source of truth. Do not place Skill runtime files at the repository root.
- Keep smoke tests, end-to-end captures, temporary fixtures, generated previews, and validation artifacts under `.ai-to-figma/` or another ignored temporary directory. Do not commit them as Skill or project content.
- Do not add a root package manifest solely to run temporary validation.

## Change requirements

- Preserve relative links inside the Skill when moving or renaming its resources.
- Keep the Chinese and English sections of `README.md` semantically aligned. Update `README.zh-CN.md` when the Chinese project instructions change.
- Validate the packaged Skill with the system `skill-creator` validator against `skills/ai-to-figma/`.
- Run any functional smoke checks from ignored local workspaces and report them separately from committed project files.

---
> Source: [Niall-Young/AItoFigma](https://github.com/Niall-Young/AItoFigma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
