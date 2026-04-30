## skills

> This repository is a lightweight collection of agent skills and helper scripts.

# AGENTS.md

This repository is a lightweight collection of agent skills and helper scripts.
There is no formal build system; most work is editing Markdown or running a
small set of Bun utilities in `skills/skill-creator/scripts/`.

## Commands (Build/Lint/Test)

### Baseline assumptions
- Bun is available on PATH (`bun`).
- No package manager config is present (no `package.json`, `pyproject.toml`, etc.).
- There is no repo-wide lint or test runner configured.

### Validation (closest thing to tests)
- Validate a skill:
  - `bun skills/skill-creator/scripts/quick_validate.ts <path/to/skill>`

### Initialize a new skill
- Create a new skill template:
  - `bun skills/skill-creator/scripts/init_skill.ts <skill-name> --path <path>`
  - Example: `bun skills/skill-creator/scripts/init_skill.ts my-new-skill --path skills/`

### Linting
- No linter is configured. Do not invent commands.
- If linting is needed, ask for preferred tool (e.g., `ruff`, `flake8`, `black`).

### Test suites
- No test runner is configured.

## Repository Structure

### Top-level
- `README.md` describes the skill collection.
- `skills/` contains individual skills and utilities.

### Skill folders
- Each skill is a directory with at least a `SKILL.md`.
- Optional subfolders: `scripts/`, `references/`, `assets/`.

### Skill creator utilities
- `skills/skill-creator/scripts/quick_validate.ts` validates `SKILL.md` frontmatter.
- `skills/skill-creator/scripts/init_skill.ts` generates a new skill template.

## Code Style Guidelines

### General
- Prefer minimal, direct changes. Avoid refactors unless requested.
- Keep scripts small and explicit; avoid over-abstraction.
- Use clear, intention-revealing names over cleverness.

### TypeScript runtime
- Target Bun (scripts use `#!/usr/bin/env bun`).
- Use Bun and Node standard library modules where possible.

### Imports
- Node and Bun imports first, then local.
- One import per line.
- Prefer `node:path` helpers over manual path concatenation.
- Avoid unused imports.

Example ordering:
```ts
import { existsSync, readFileSync } from "node:fs";
import { join, resolve } from "node:path";

import { validateSkill } from "./quick_validate";
```

### Formatting
- Use 2-space indentation.
- Keep line length reasonable (~100 chars); wrap long strings if needed.
- Use blank lines between top-level functions.
- Use concise comments only when behavior is non-obvious.

### Naming conventions
- Functions and variables: `camelCase`.
- Constants: `UPPER_SNAKE_CASE`.
- Classes (if introduced): `PascalCase`.
- Skill identifiers: `hyphen-case` (lowercase letters, digits, hyphens).

### Paths and files
- Use `node:path` for filesystem paths.
- Avoid string concatenation for paths; use `join` or `resolve`.
- Always resolve user-provided paths with `resolve(...)`.

### Error handling
- Prefer early returns with clear user-facing messages.
- Use `try/catch` around filesystem operations.
- Return `null` or `false` to signal failure; avoid throwing new exceptions
  unless required.

### CLI behavior
- Provide usage help on incorrect arguments.
- Exit with non-zero status on failure.
- Use concise, user-readable output.

### Strings and output
- Prefer template literals for interpolation.
- Keep messages short; include context (file path, skill name).

### Data validation
- Validate required fields early.
- Keep validation rules simple and explicit.
- Do not silently coerce invalid values.

## Markdown Style (SKILL.md files)

### Frontmatter
- Must start with `---` and include YAML frontmatter.
- Required keys: `name`, `description`.
- Allowed keys: `name`, `description`, `license`, `allowed-tools`, `metadata`.

### Naming rules for `name`
- Must be `hyphen-case`.
- No leading/trailing hyphen or consecutive `--`.
- Max length 64 characters.

### Description rules
- Must be plain text; no angle brackets.
- Max length 1024 characters.

### Sections
- Use clear, task-oriented headings.
- Provide concrete examples when relevant.

## Tooling notes for agents

### What to edit
- Most changes are in `SKILL.md` files and related references.
- Keep edits localized to the skill you are working on.

### What not to do
- Do not invent lint/test commands.
- Do not add new files unless required.
- Do not add new dependencies without approval.

---
> Source: [jmmarotta/skills](https://github.com/jmmarotta/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
