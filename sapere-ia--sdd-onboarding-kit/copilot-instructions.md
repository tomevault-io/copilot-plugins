## sdd-onboarding-kit

> This repository is an SDD onboarding kit. It is not the target project configuration.

# Instructions for Claude Code when working inside this kit

This repository is an SDD onboarding kit. It is not the target project configuration.

When the developer asks to install or configure SDD in another repository:

1. Start by reading `instructions.md`.
2. Then read `questions.md`.
3. Then read `output-project-structure.md`.
4. Use the files under `templates/`, `agents/`, `skills/`, `hooks/`, `mcps/`, and `scripts/` as source material.
5. Do not treat this kit's root `CLAUDE.md` as the final project `CLAUDE.md`.
6. Generate or update the target project's own `CLAUDE.md` from `templates/CLAUDE.md.template`.
7. Ask the developer before enabling hooks, configuring MCPs, changing git policy, or assuming test/lint commands.

The final goal is to create a project-specific SDD harness, not to explain SDD abstractly.

When the developer asks to make changes in this repository:
1. If `todolist.md` exists (it is gitignored and local-only), read it: it contains the tasks to be done. If it does not exist, ask the developer what to change.
2. Complete the tasks. You may need to modify some of the hooks, MCPs, templates, etc.
3. If `bugs.md` exists (also gitignored and local-only), check that the bugs it describes are solved.
4. Mark with X all the checkboxes of the tasks/bugs you complete.
5. After adding, renaming or deleting kit files, run `scripts/update-manifest.sh` to regenerate `manifest.md`.

---
> Source: [Sapere-IA/sdd-onboarding-kit](https://github.com/Sapere-IA/sdd-onboarding-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
