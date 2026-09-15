## use-uv

> Always use uv for Python environments, installs, and script execution


# Use uv for Python

Always use `uv` for Python work in this repo. Do not use bare `pip`, `pip3`, `python -m pip`, or `python3 -m venv`.

## Rules

- Run scripts / one-offs with `uv run` (prefer `uv run --no-project` for throwaway tooling that should not resolve the repo environment).
- Add or sync project deps with `uv add` / `uv sync`, not `pip install`.
- Create virtualenvs with `uv venv`, not `python -m venv`.
- Prefer `uvx` for ephemeral CLI tools.
- If a command needs extra packages outside the project, use `uv run --with <pkg>` (and `--no-project` when isolation is wanted).

## Examples

```bash
# good
uv run python scripts/foo.py
uv run --no-project --with paramiko --with pyyaml python -c '...'
uv sync
uv add numpy

# bad
pip install numpy
python3 -m pip install -r requirements.txt
python3 -m venv .venv
```

---
> Source: [djmango/openfront-ai](https://github.com/djmango/openfront-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
