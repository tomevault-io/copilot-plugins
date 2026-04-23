## coffee-milk-beer

> Use `uv` for all Python dependency, environment, and script management in this repository.

## Python dependency and environment management: uv

Use `uv` for all Python dependency, environment, and script management in this repository.

Project root: [pyproject.toml](mdc:pyproject.toml)

### Conventions
- **Environment**: Do not commit or rely on `venv`/`pip`. Use `uv` exclusively.
- **Lockfile**: Commit `uv.lock`. Regenerate via `uv lock` when adding/removing deps.
- **Testing**: Prefer `pytest` and run it via `uv`.
- **Scripts**: Prefer `uv run ...` to ensure the project environment is active.

### Common commands
- **Install uv** (if not present):
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

- **Sync environment from lockfile**:
```bash
uv sync
```

- **Add a dependency** (default: application dependency):
```bash
uv add <package>
```

- **Add a dev-only dependency** (e.g., pytest, ruff):
```bash
uv add --dev <package>
```

- **Remove a dependency**:
```bash
uv remove <package>
```

- **Update the lockfile** (without changing pinned versions):
```bash
uv lock
```

- **Upgrade all direct dependencies** (refresh pins):
```bash
uv lock -U
```

- **Run a command within the project environment**:
```bash
uv run <command> [args]
# examples
uv run python tools/generate_isochrones.py
uv run python tools/convert_to_pmtiles.py
```

- **Run tests with pytest**:
```bash
uv run pytest -q
```

- **See dependency tree**:
```bash
uv tree
```

- **Export lockfile to requirements.txt** (if needed for external tooling):
```bash
uv export --format requirements.txt --output requirements.txt --locked
```

### Python versions
Prefer managing Python via `uv`:
```bash
uv python install <version>
uv python list
uv python pin <version>
```
If a Python version is pinned in the project, `uv sync` will respect it.

### Inline script metadata (PEP 723)
For one-off scripts, `uv` supports inline dependencies. Example:
```bash
# Add a dependency to a script
uv add --script path/to/script.py requests

# Run the script in an isolated env
uv run path/to/script.py
```

### CI and bots
- **CI** should use `uv sync` before running Python tasks/tests.
- **Dependabot** can update `uv.lock`:
```yaml
dependabot.yml: |
  version: 2
  updates:
    - package-ecosystem: "uv"
      directory: "/"
      schedule:
        interval: "weekly"
```

### Notes
- Always prefer `uv run ...` over activating a venv manually.
- When adding new tools, add them with `uv add --dev` and document usage in `README.md` if user-facing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PhilipMathieu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
