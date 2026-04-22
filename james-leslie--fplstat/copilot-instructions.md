## fplstat

> - Use `uv` for ALL Python operations (no `pip`, `python`, or `poetry`)

# Development Environment

## Python

- Use `uv` for ALL Python operations (no `pip`, `python`, or `poetry`)
- ALWAYS use `uv run` prefix for Python scripts
- Do NOT use bare `python` or `pip` commands

### `uv` CLI Common Commands

- Run scripts: `uv run <script.py>`
- Add dependencies: `uv add <package>`
- Add dev dependencies: `uv add --dev <package>`
- Remove dependencies: `uv remove <package>`
- Sync dependencies: `uv sync`

Wherever possible, use the uv CLI for all operations, rather than editing `pyproject.toml` directly.

> Reference: [uv Documentation](https://docs.astral.sh/uv/)

## Documentation

Zensical, and all necessary plugin dependencies, are installed as dev dependencies and managed via `uv`.

The Zensical configuration is in @zensical.toml. This file defines site settings, theme configuration, navigation structure, and plugins.

> Reference: [Zensical Documentation](https://zensical.org/docs)

## Markdown

- Format markdown files: `uv run mdformat <file.md>`
- Check without modifying: `uv run mdformat --check <file.md>`
- Configuration is in `.mdformat.toml`

> Reference: [mdformat Documentation](https://mdformat.readthedocs.io/)

# Workflow

IMPORTANT: Follow this sequence for ALL development tasks

## 1. Plan Mode

- Look for existing issues or PRs that may relate: `gh issue list` and `gh pr list`
- Think through edge cases and architecture
- Ask questions if unclear
- Break work into small, manageable tasks
- Present a clear plan

## 2. Implementation

- Create a new issue if none exists: `gh issue create`
- ALWAYS create a new branch first: `git checkout -b <branch-name>`
- Bump package version using `uv version patch` (or `minor`/`major` as needed)
- Work through plan items sequentially
- After completing each item:
    - Ensure Python code is formatted: `uv run ruff format <file>`
    - Ensure markdown is formatted: `uv run mdformat <file.md>`
    - Run tests if applicable: `uv run pytest <relevant_tests>`
    - Create a focused commit: `git add <files> && git commit -m "<clear message>"`
    - Repeat until all items complete

## 3. Finalization

- Review and update @docs if needed
- Run full test suite: `uv run pytest`
- Push branch: `git push origin <branch-name>`
- Open a Pull Request for review: `gh pr create`

## Git Practices

- Branch naming: `feature/<description>` or `fix/<description>`
- Commit messages: Clear, present tense, describe what and why
- NEVER commit directly to main/master
- Use the GitHub CLI whenever necessary

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/James-Leslie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
