## project

> You are working on devopsdaysdc25, a Python application built with modern best practices.

type: always

---

# devopsdaysdc25 Project Rules

You are working on devopsdaysdc25, a Python application built with modern best practices.

## Project Info
- Description: Let's vibe 🚀
- Company: jonzeolla
- Python 3.13+
- Package manager: uv (NOT pip/poetry)

## Tech Stack
- Testing: pytest (>80% coverage)
- Linting: ruff, pyright, refurb
- Security: grype, syft
- Docker: Multi-platform builds
- CI/CD: GitHub Actions

## Code Requirements
- Max line: 120 chars
- Type hints required
- Google-style docstrings
- pathlib only (no os.path)
- logging only (no print)
- Specific exceptions

## Key Commands
```bash
task init          # Setup
task build test    # Before commits
task docker-build  # Build container
task release       # Release
```

## Important
Look for `NotImplementedError` markers - implement these with business logic.

---
> Source: [JonZeolla/devopsdaysdc25](https://github.com/JonZeolla/devopsdaysdc25) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
