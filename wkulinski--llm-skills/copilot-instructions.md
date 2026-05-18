## llm-skills

> This repository is a curated set of LLM skills. Keep changes scoped to the relevant skill directory.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a curated set of LLM skills. Keep changes scoped to the relevant skill directory.

- `.agents/skills/<skill-name>/SKILL.md`: primary instructions for a skill.
- `.agents/skills/<skill-name>/scripts/*.sh`: optional automation helpers used by that skill.
- `README.md`: high-level setup notes.
- `.env.dist`: template for local environment variables.
- `var/`: local cache/state (ignored by git; do not treat as source).

Use `kebab-case` for new skill directories (for example, `my-new-skill`).

## Build, Test, and Development Commands
There is no single build system; use lightweight validation commands:

- `cp .env.dist .env.local`: initialize local config.
- `rg --files .agents/skills`: quickly list tracked skill files.
- `find .agents/skills -type f -path '*/scripts/*.sh' -print0 | xargs -0 -n1 bash -n`: syntax-check shell scripts.
- `find .agents/skills -type f -path '*/scripts/*.sh' -print0 | xargs -0 -n1 shellcheck`: lint shell scripts (if installed).

If you add a script, ensure it is executable: `chmod +x .agents/skills/<skill>/scripts/<name>.sh`.

## Coding Style & Naming Conventions
- Shell scripts: start with `#!/usr/bin/env bash` and `set -euo pipefail`.
- Prefer small, composable scripts with explicit `usage()` for CLI-style entrypoints.
- Keep paths repo-relative and configurable (avoid hardcoded machine-specific paths).
- Markdown should use clear headings, short sections, and actionable steps.

Match the language and tone of the existing skill when extending it.

## Testing Guidelines
- For script changes, run at least `bash -n` before opening a PR.
- Smoke-test behavior with a minimal invocation (for example, `--help` or a no-op mode).
- For docs-only updates, verify referenced paths/commands exist.

When adding a skill, include at least one concrete command example in `SKILL.md`.

## Commit & Pull Request Guidelines
This branch currently has no commit history, so no existing commit convention can be inferred. Use clear, conventional messages:

- `docs(skills): add contributor notes for qa-run`
- `fix(code-implement): guard missing state file`

PRs should include:
- a short problem/solution summary,
- list of changed skill paths,
- validation commands you ran and outcomes,
- linked issue (if applicable).

## Security & Configuration Tips
- Never commit secrets (`GH_TOKEN`, local `.env.local` values).
- Resolve tool entrypoints through `.agents/skills/_shared/scripts/env-load.sh` (`resolve_tool_cmd`) instead of composing paths manually.
- Keep generated cache/state under `CACHE_PATH`/`var/` out of version control.

## Documentation path map contract
Skills that read/write project documentation use a path map from the **consumer project** `AGENTS.md`.

Minimal example (for this repo):
```yaml
docs_map:
    MAIN_DOC: docs/README.md
    HANDOFF_DOC: var/agent/HANDOFF.md
    SKILLS_INDEX_DOC: docs/SKILLS.md
```

Notes:
- This is a minimal map, not a full matrix for every skill.
- In target projects define only keys required by skills you plan to run.
- Missing required key should block only that skill and trigger a question for the missing path.

### Documentation path map contract (project `AGENTS.md`)
Skills that read/write project documentation use a path map from the **consumer project** `AGENTS.md`.

Use this block in project `AGENTS.md`:
```yaml
docs_map:
    MAIN_DOC: docs/README.md
    AGENT_RULES_DOC: AGENTS.md
    COMMIT_MESSAGE_DIR: /tmp/
    HANDOFF_DOC: var/agent/HANDOFF.md
    SKILLS_INDEX_DOC: docs/SKILLS.md
```

---
> Source: [wkulinski/llm-skills](https://github.com/wkulinski/llm-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
