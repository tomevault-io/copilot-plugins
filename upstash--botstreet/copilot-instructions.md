## botstreet

> This file is for local development only. Ahi does not upload `AGENTS.md` to remote boxes.

# Ahi Development Instructions

This file is for local development only. Ahi does not upload `AGENTS.md` to remote boxes.

## Purpose

Use this file to understand how to work on an Ahi project as a coding agent.
The deployed runtime behavior lives in the path configured by `skills:` in `ahi.yaml`, typically `skills/SKILL.md`.

## Source Of Truth

- `ahi.yaml` defines the agents, models, setup commands, and schedules.
- `skills/SKILL.md` defines deployed runtime behavior.
- `AGENTS.md` defines local development workflow for coding agents.
- `tools/` contains executable project tools.
- `data/` is project-local state during development. Remote runtime state lives in each box.

## Workflow

- Read `ahi.yaml` before making changes.
- Start with `skills/SKILL.md` to understand the runtime behavior you are changing.
- Read other files under `skills/` when the task depends on a specific workflow package.
- Use scripts in `tools/` instead of inventing replacement commands.
- Test changes locally with `ahi dev` before using `ahi apply`.
- Use `ahi apply` for code, skill, config, env, setup, and schedule changes.
- Use `ahi pull-data` or `ahi push-data` only when you intentionally want to transfer runtime data between local and remote.

## Boundaries

- Do not assume `AGENTS.md` is available inside the remote box.
- Do not treat local `data/` as automatically synchronized with any remote agent.
- Do not change `skills/SKILL.md` unless you intend to change deployed runtime behavior.

---
> Source: [upstash/botstreet](https://github.com/upstash/botstreet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
