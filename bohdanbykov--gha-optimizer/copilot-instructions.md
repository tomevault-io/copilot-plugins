## gha-optimizer

> list of files AI agent should review and update if needed


For EVERY task, you must review these files and update them if your changes affect them:

CORE FILES (always check):
- pyproject.toml (dependencies, version, metadata)
- src/gha_optimizer/__init__.py (if version/exports change)

DOCUMENTATION FILES (check and update if affected):
- README.md (main user-facing guide)
- CONTRIBUTING.md (development process)
- INSTALL.md (installation steps)
- docs/ (any technical documentation)

CI/CD FILES (check and update if affected):
- .github/workflows/ci.yml
- .github/workflows/release.yml
- .pre-commit-config.yaml (must match ci.yml tools)

PACKAGING FILES (check and update if affected):
- MANIFEST.in (file inclusion rules)
- test_cli.py (integration tests)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BohdanBykov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
