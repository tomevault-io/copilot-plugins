## git-workflow

> Git workflow and CI/CD rules for ecommerce_recommendator


# Git Workflow Rules

## CI Requirements

- **NEVER merge or push to main if CI is failing**
- Always wait for CI checks to pass before merging PRs
- If CI fails, fix the issues first - no exceptions

## Before Pushing

1. Run locally first:
   ```bash
   ruff check . && ruff format --check . && mypy . && pytest
   ```
2. Only push if all checks pass locally

## Branch Protection (Configured)

Branch protection is enabled on `main` with required status checks:
- Lint (Ruff)
- Type Check (mypy)
- Test (pytest)
- Security Scan
- All Checks Passed

GitHub will block merges if any check fails.

## PR Workflow

1. Create feature branch: `git checkout -b feat/feature-name`
2. Make changes with semantic commits
3. Push and create PR
4. Wait for CI to pass (lint, type-check, test, security)
5. Only merge after ALL checks are green

---
> Source: [Plenix-AI/ecommerce_recommendator](https://github.com/Plenix-AI/ecommerce_recommendator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
