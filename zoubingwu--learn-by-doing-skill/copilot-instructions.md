## learn-by-doing-skill

> This repository contains the `learn-by-doing` agent skill. The skill guides coding agents to help users learn programming by generating real local challenge projects, tests, feedback, and progress state.

# AGENTS.md

## Project Purpose

This repository contains the `learn-by-doing` agent skill. The skill guides coding agents to help users learn programming by generating real local challenge projects, tests, feedback, and progress state.

## Architecture

- `skills/learn-by-doing/SKILL.md`: core agent workflow, trigger metadata, contracts, and runtime rules.
- `skills/learn-by-doing/agents/openai.yaml`: UI metadata for the skill.
- `skills/learn-by-doing/scripts/detect_environment.py`: Python standard-library tool probe used by agents after they infer the required tool list.
- `skills/learn-by-doing/scripts/state.py`: Python standard-library state manager for `.learn-by-doing/state.json`.
- `skills/learn-by-doing/scripts/render_dashboard.py`: Python standard-library renderer for `.learn-by-doing/dashboard.html`.
- `skills/learn-by-doing/references/challenge-design.md`: detailed challenge and stage design guidance.
- `skills/learn-by-doing/references/feedback-policy.md`: feedback, hinting, repair, and installation-confirmation guidance.
- `tests/`: standard-library `unittest` coverage for scripts.
- `docs/superpowers/specs/`: approved design specs.
- `docs/superpowers/plans/`: implementation plans.

## Development Rules

- Keep the skill generic. Runtime intelligence belongs in the agent instructions.
- Keep scripts small, deterministic, dependency-free, and cross-platform.
- Use Python standard library for scripts and tests.
- Use English for skill files, script comments, references, and field descriptions.
- Update this file when the architecture or directory structure changes.
- Commit only after an explicit user request.

---
> Source: [zoubingwu/learn-by-doing-skill](https://github.com/zoubingwu/learn-by-doing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
