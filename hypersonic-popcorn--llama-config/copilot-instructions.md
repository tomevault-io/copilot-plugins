## llama-config

> PY_VER=$(cat .python-version | tr -d '[:space:]'); MAJOR=$(echo $PY_VER | cut -d. -f1); MINOR=$(echo $PY_VER | cut -d. -f2); uv run black --target-version "py${MAJOR}${MINOR}" .

## How to run

```
uv run pytest
PY_VER=$(cat .python-version | tr -d '[:space:]'); MAJOR=$(echo $PY_VER | cut -d. -f1); MINOR=$(echo $PY_VER | cut -d. -f2); uv run black --target-version "py${MAJOR}${MINOR}" .
uv run basedpyright
```

- Wrap lines at 88 characters (per `.flake8` and `[tool.black]` line-length).
- Before any `git stash`, `commit`, or `merge`, verify:
  - All tests pass (`uv run pytest`)
  - Coverage is at least 80% (`uv run pytest --cov=src --cov-report=term-missing`)
  - Code is formatted (`uv run black --target-version "py312" .`)
  - Type checking passes (`uv run basedpyright`)

## Single command start

To start backend + frontend together from any directory:

```bash
uv run app
```

Or from the project root:

```bash
uv run src.app
```

Use `-h`/`--help` to see available arguments for overriding settings.
`llama-config.yaml` is created in the current directory on first run and
persists your settings between invocations.

## Architecture

For frontend,read and follow: @plan_frontend.md
For the backend, read and follow: @plan_backend.md

## Gotchas

- **Package manager**: use `uv` — not pip, poetry, or pdm. Run `uv sync` after pulling.
- **No conftest.py** — all fixtures are defined in individual test files.
- **Follow the .python-version for python syntax**
- **Git workflow**: when you ask me to "merge", "stage, commit, and merge", or similar, I will automatically handle all three steps — staging relevant files, committing with a descriptive message, and merging into the target branch. No need to say "please stage" or "go ahead".
- **Never commit directly to main** — create a feature branch (e.g. `feat/foo`), commit there, then merge to main when complete. No need to create PRs on GitHub.
- **Reference config**: `config_example.yaml` is a real llama-swap config — use it to understand the actual YAML format and nested keys.

## Nono Sandbox

When a file or directory access fails, run:

    nono why --self --path <path> --op <read|write|readwrite> --json

Present the result to the user and ask them to update the nono script before retrying.

If the set of paths a new tool or command needs is unknown, suggest the user run:

    nono learn --profile opencode -- <command>

This traces the command and shows what paths would need to be allowed. The user
must update the nono script themselves — never attempt to modify the sandbox
configuration directly.

## Frontend Development

- **Accessing the app**: Available at `http://localhost:5173/` and via Tailscale at `http://100.78.106.18:5173/`.
- **Errors**: If the user reports a blank page, ask them to check the browser devtools console for JavaScript errors.

## Coding conventions

When working with Python code, read and follow: @rules/python.md
When working on the frontend read and follow: @rules/frontend.md
When writing tests, read and follow: @rules/testing_practices.md
Nothing in the local_test directory should be staged or committed to git.

---
> Source: [Hypersonic-Popcorn/llama-config](https://github.com/Hypersonic-Popcorn/llama-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
