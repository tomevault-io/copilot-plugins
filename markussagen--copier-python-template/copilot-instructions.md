## copier-python-template

> This document provides instructions for working with the copier-python-template template itself.

# Claude Code Instructions for copier-python-template Template

This document provides instructions for working with the copier-python-template template itself.

## Project Overview

**copier-python-template** is a modern Python project template that generates projects following the Arkalos directory structure. It combines best practices from copier-uv with a comprehensive, production-ready setup.

## Template Structure

```
copier-python-template/
├── copier.yml              # Template configuration and questions
├── extensions.py           # Jinja2 custom filters and extensions
├── project/                # Template files (all .jinja suffixed)
│   ├── app/               # Arkalos app structure
│   ├── config/            # Configuration templates
│   ├── tests/             # Test templates
│   ├── docs/              # Documentation templates
│   ├── .github/           # GitHub Actions workflows
│   ├── pyproject.toml.jinja
│   ├── Makefile.jinja
│   ├── Dockerfile.jinja
│   ├── compose.yaml.jinja
│   ├── .mise.toml.jinja
│   ├── .pre-commit-config.yaml.jinja
│   └── CLAUDE.md.jinja
├── CLAUDE.md              # This file
└── readme.md              # Template documentation
```

## Modifying the Template

### Adding New Questions

Edit `copier.yml`:

```yaml
new_option:
  type: bool
  help: Enable new feature?
  default: false
```

### Adding Conditional Features

Use Jinja2 conditionals in template files:

```jinja2
{%- if new_option %}
# Feature-specific code
{%- endif %}
```

### Adding Dependencies

In `pyproject.toml.jinja`:

```jinja
{%- if new_option %}
[project.optional-dependencies]
new_feature = [
    "package>=1.0.0",
]
{%- endif %}
```

### Adding Directories

Use conditional directory names:

```
project/app/{% if new_option %}new_feature{% endif %}/
```

## Testing the Template

### Generate a Test Project

```bash
# From parent directory
copier copy --trust copier-python-template test-project

# Answer prompts, then test the generated project
cd test-project
make setup
make lint
make test
```

### Test with All Options

```bash
# Create a test script to try all combinations
copier copy --trust \
  -d project_name="Test Project" \
  -d include_http=true \
  -d include_ai=true \
  -d include_docker=true \
  copier-python-template test-all-features
```

## Template Features

### Core Features (Always Included)

- **Build system**: Hatchling + hatch-vcs for git-based versioning
- **Package manager**: uv for fast, reliable dependency management
- **Tool manager**: mise for Python version management
- **Code quality**: Ruff (linting + formatting)
- **Type checking**: ty (fast) + mypy (comprehensive)
- **Testing**: pytest with coverage
- **Pre-commit**: prek (Rust-based, faster than pre-commit)
- **Documentation**: MkDocs Material
- **CI/CD**: GitHub Actions
- **Structure**: Arkalos project layout

### Optional Features (User-Selected)

- **HTTP/Web** (`include_http`):
  - FastAPI setup with routes/controllers/middleware
  - Health check endpoint
  - Docker support
  - API testing fixtures

- **AI/ML** (`include_ai`):
  - AI directories (agents, actions, evals, trainers)
  - ML dependencies (numpy, scikit-learn)
  - AI dependencies (openai, anthropic)

- **Data Processing** (`include_data_processing`):
  - Data directories (extractors, transformers, analyzers, visualizers)
  - Data dependencies (pandas, pyarrow, polars)

- **Hardware** (`include_hardware`):
  - Hardware directories (sensors, actuators, controllers, communicators)

- **Docker** (`include_docker`):
  - Multi-stage Dockerfile
  - docker-compose.yaml
  - .dockerignore

- **DevContainer** (`include_devcontainer`):
  - VS Code devcontainer configuration
  - Pre-configured extensions
  - Automatic environment setup

- **CLI** (`python_package_command_line_name`):
  - Typer-based CLI setup
  - Entry point configuration

## Jinja2 Extensions

Located in `extensions.py`:

### GitExtension

Filters to auto-detect git configuration:

```jinja2
{{ 'Default Name' | git_user_name }}
{{ 'default@email.com' | git_user_email }}
```

### SlugifyExtension

Convert strings to URL-safe slugs:

```jinja2
{{ project_name | slugify }}        # my-project
{{ project_name | slugify('_') }}   # my_project
```

### CurrentYearExtension

Global variable for current year:

```jinja2
{{ current_year }}  # 2026
```

## Best Practices for Template Development

### 1. Use Conditional Directories

Don't create directories for features the user didn't select:

```
app/{% if include_http %}http{% endif %}/
```

### 2. Use Optional Dependencies

Keep core dependencies minimal, make feature-specific deps optional:

```toml
[project.optional-dependencies]
web = ["fastapi[standard]>=0.115.0"]
```

### 3. Provide Good Defaults

Auto-fill derived values:

```yaml
python_package_import_name:
  default: "{{ project_name | slugify('_') }}"
```

### 4. Test All Combinations

Test with different feature combinations to ensure they work together.

### 5. Document Everything

- Update CLAUDE.md for generated projects
- Update README.md for the template itself
- Add comments in complex Jinja2 templates

## Updating from copier-uv

This template is based on copier-uv but with significant modifications:

### Key Differences

| Feature | copier-uv | copier-python-template |
|---------|-----------|----------------|
| Layout | src/ | app/ (Arkalos) |
| Command runner | duty (Python) | Makefile |
| Build backend | PDM | Hatchling + hatch-vcs |
| Ruff config | config/ruff.toml | pyproject.toml |
| Type checker | mypy only | ty + mypy |
| Tool manager | None | mise |
| Pre-commit | pre-commit | prek |
| Docker | Not included | Multi-stage build |
| DevContainer | Not included | VS Code devcontainer |
| Structure | Minimal | Modular Arkalos |

### Updating Template

To incorporate new features from copier-uv:

1. Check copier-uv releases
2. Review changes
3. Adapt to Arkalos structure
4. Test thoroughly
5. Update CHANGELOG.md

## Contributing to the Template

### Making Changes

1. Clone the repository
2. Make changes to template files
3. Test with `copier copy`
4. Verify generated project works
5. Update documentation

### Release Process

1. Update version in README.md
2. Update CHANGELOG.md
3. Create git tag: `git tag v0.1.0`
4. Push: `git push origin v0.1.0`

## Troubleshooting

### Generated Project Issues

**Missing dependencies**:
- Check conditional logic in pyproject.toml.jinja
- Ensure optional dependencies are properly defined

**Missing directories**:
- Check conditional directory names
- Ensure Jinja2 conditions match copier.yml options

**Tests failing**:
- Verify test templates match project structure
- Check imports in test files

### Template Issues

**Copier errors**:
- Validate copier.yml syntax
- Check Jinja2 template syntax
- Ensure all referenced variables exist

**Jinja2 rendering errors**:
- Check filter usage
- Verify extension imports in copier.yml
- Test with simple values first

## Python Development Standards

### 1. Import Structure & Typing

We strictly follow this import order to maximize readability and performance.

**Template:**

```python
# 1. Future imports (REQUIRED)
from __future__ import annotations

# 2. Standard Library (Explicit imports preferred)
import pathlib
import datetime
import typing as T
import logging

# 3. Third-Party (Using 'from' is acceptable)
from pydantic import BaseModel
import pandas as pd

# 4. Local / Application Imports
from app.core.config import settings
from app.utils import formatting

# 5. Type Checking Block (REQUIRED for type-only imports)
if T.TYPE_CHECKING:
    from app.services.model_loader import ModelLoader
    from app.db.client import DatabaseClient

# Code starts here...
```

##### Why Explicit Standard Imports?

- The reader knows exactly where the object comes from.
    - `import datetime` -> `datetime.datetime.now()`

- Ambiguous (is it the module or class?).
    - `from datetime import datetime -> datetime.now()`

Makes refactoring easier (grep finds usages reliably).

### 2. Environment & Package Management

- Manager: We use uv (or poetry) for dependency management.
- Run: Always prefix commands with the environment runner (e.g., uv run python main.py).
- Lockfile: uv.lock / poetry.lock is the single source of truth for dependencies.

### 3. Configuration

- Twelve-Factor Principles: Config is stored in environment variables, not code.
- Centralized Settings: Use pydantic-settings to validate config in one place.
- No Hardcoded Paths: Never use string paths. Use pathlib.Path relative to a configured root.

Bad:

```python
MODEL_PATH = "/usr/local/models/v1/model.bst" # Hardcoded
```

```python
# app/config.py
class Settings(BaseSettings):
    base_model_dir: pathlib.Path
    environment: str = "development"

settings = Settings()

# Usage
model_path = settings.base_model_dir / "v1" / "model.bst"
```

## Modifiability Principles

### 1. Abstraction of External Dependencies

Isolate all I/O. This is the number one rule for testability.

```python
# DON'T - Tightly coupled to filesystem and specific library
def load_model():
    with open("models/model.pkl", "rb") as f:
        return pickle.load(f)

# DO - Dependency Injection & Abstraction
class ModelLoader:
    def load(self, path: pathlib.Path) -> ModelArtifact:
        """Abstraction over loading mechanics."""
        ...

def run_prediction(loader: ModelLoader, path: pathlib.Path):
    model = loader.load(path)
    ...
```

Benefits:

- You can mock ModelLoader in tests without creating fake files.
- You can swap pickle for safetensors or onnx without changing the business logic calling it.

### 2. Pure Functions Where Possible

```python
# Pure (Easy to test)
def calculate_score(features: dict, weights: list) -> float:
    return sum(f * w for f, w in zip(features.values(), weights))

# Impure (Hard to test)
def calculate_and_save(features: dict):
    weights = db.get_weights() # Hidden dependency
    score = sum(...)
    db.save(score) # Side effect
```

### 3. Dependency Injection

Pass dependencies into functions/classes rather than importing global instances.

```python
# DON'T
from app.db import database
def get_user(id):
    return database.query(id)

# DO
def get_user(id: str, db: DatabaseClient):
    return db.query(id)
```

## Testing Philosophy

- Code is the source of truth. If the code works and the test fails, and the test is testing an implementation detail (like a specific log message or internal variable), delete the test.
- Test Public Interfaces. Do not test private methods (_method). Test the input and output of the public API.
- Use tmp_path. For any filesystem tests, use the pytest tmp_path fixture. Never write to the actual repository structure during tests.
- Mock Externalities. If it talks to the internet or a database, mock it.
- Make sure to test docstrings

## The Twelve-Factor App (ML Adaptation)

- Codebase: One repo, many deploys (dev, staging, prod).
- Dependencies: Explicitly declared in pyproject.toml. No system-wide packages.
- Config: Strictly env vars. No secrets in Git.
- Backing Services: databases, buckets, and queues are addressable resources via URL/connection strings.
- Build, Release, Run: Strictly separate stages. Docker images are immutable.
- Processes: Stateless. Any state must be in a backing service (Redis, DB), not in Python memory between requests.
- Dev/Prod Parity: Keep dev as close to prod as possible. Use Docker locally if needed.

## Engineering Guidelines & Philosophy

### Core Directives

These are the non-negotiable standards for this codebase.

- **DO include `from __future__ import annotations`** as the very first import in every Python file (except `__init__.py`). This enables modern type hinting and cleaner forward references.
- **DO include `if TYPE_CHECKING:` blocks** for imports used solely for type hinting. This prevents circular imports and reduces runtime startup overhead.
- **DO minimize dependencies.** If a standard library solution exists (e.g., `pathlib`, `json`, `argparse`), use it instead of adding a heavy third-party package.
- **DO streamline workflows.** If a task requires more than two terminal commands, wrap it in a `justfile` (or Makefile) command.
- **DO be pragmatic.** Solve for the "here and now." Do not over-engineer for hypothetical future use cases. However, write code that is modular enough to be extended when that future arrives.

---

### Engineering Philosophy

#### Code is Read 10x More Than Written

- **Optimize for modifiability** over cleverness.
- **Explicit is better than implicit.** Code should be self-documenting.
- **Pragmatic over dogmatic.** If tests are preventing a critical hotfix, remove the tests and rewrite them later.
- **Code is the source of truth, not tests.** Tests serve the code to provide confidence; the code does not serve the tests.
- **Abstraction layers enable change.** Isolate external dependencies (network, filesystem, APIs, databases).

#### The Pragmatic Approach

Think as a pragmatic CTO or Staff Engineer:
- **Production-First:** Code should be robust, logged, and observable.
- **Infrastructure-Aware:** Understand how the code runs in Docker/K8s.
- **Trade-off Oriented:** Make decisions that optimize for long-term maintainability without stalling short-term progress.

---

## Version Control

### Commit Messages

Follow Conventional Commits:

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Test changes
- `chore`: Maintenance tasks

Example: `feat: add user authentication endpoint`

### Branching

- Main branch: `main`
- Feature branches: `feature/description`
- Bug fixes: `fix/description`

## Documentation

Documentation is built with MkDocs Material:

```bash
make docs        # Serve locally at http://localhost:8000
make docs-build  # Build static site
make docs-deploy # Deploy to GitHub Pages
```

- Add markdown files to `docs/`
- API reference is auto-generated from docstrings
- Update `mkdocs.yml` for navigation changes

## CI/CD

GitHub Actions workflows:

- **CI** (`.github/workflows/ci.yml`):
  - Linting (ruff)
  - Type checking (ty + mypy)
  - Tests (pytest) across Python versions and OS
  - Docker build validation

- **Release** (`.github/workflows/release.yml`):
  - Triggered on git tags (`v*`)
  - Builds and publishes to PyPI
  - Creates GitHub release

## Troubleshooting

### Common Issues

**Import errors**:
- Ensure you're using `uv run` to run scripts
- Check `pythonpath = ["."]` in pyproject.toml

**Type checking errors**:
- Run `uv run ty check app tests` for fast feedback
- Run `uv run mypy app tests` for comprehensive checks

**Test failures**:
- Tests stop on first failure (`--maxfail=1`)
- Run `make test-cov` to see coverage report

## Additional Resources

- [Copier documentation](https://copier.readthedocs.io/)
- [Jinja2 documentation](https://jinja.palletsprojects.com/)
- [Arkalos structure](https://github.com/arkalos-org/arkalos)
- [copier-uv (base template)](https://github.com/pawamoy/copier-uv)
- [uv documentation](https://docs.astral.sh/uv/)
- [ty type checker](https://astral.sh/blog/ty)
- [Ruff documentation](https://docs.astral.sh/ruff/)
- [FastAPI documentation](https://fastapi.tiangolo.com/)
- [pytest documentation](https://docs.pytest.org/)
- Modern Python project structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarkusSagen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
