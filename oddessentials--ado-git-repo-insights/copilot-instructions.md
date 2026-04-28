## ado-git-repo-insights

> - Python 3.12+ (backend, scripts, tests)

# ado-git-repo-insights Development Guidelines

## Active Technologies
- Python 3.12+ (backend, scripts, tests)
- TypeScript 6.x (extension UI, tests)
- pandas (aggregation), pytest (Python tests), Jest (TypeScript tests)
- ruff (Python linting), ESLint (TypeScript linting), mypy (type checking)
- esbuild (IIFE bundler), Playwright (smoke tests)

## Project Structure

```text
src/
tests/
```

## Commands

```bash
python scripts/run_pytest.py              # Python tests (coverage-safe launcher)
cd extension && pnpm test                 # Extension tests
python scripts/run_pr_preflight.py        # Authoritative local PR gate
```

## Code Style

Python 3.12+ (backend), TypeScript 6.x (frontend): Follow standard conventions

## Recent Changes
- 052-review-time-pipeline: Added Python 3.12+ (backend pipeline), TypeScript 6.x (extension UI — no changes needed) + pandas (aggregation), requests (ADO API client), sqlite3 (persistence), pytest (testing)
- 049-cross-platform-hardening: Replaced PowerShell ACL check with Python, build-demo.sh with build_demo.py, added SETUP/INFRA/GATE error categories, Node engine enforcement
- 048-https-github-com: mypy strict mode on src/ with disallow_any_generics = true

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oddessentials) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
