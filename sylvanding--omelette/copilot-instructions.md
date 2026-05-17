## git-conventions

> Git commit conventions and workflow rules


# Git Conventions

## Commit Format (Conventional Commits, enforced by pre-commit)
```
<type>(<scope>): <description>

[optional body]
```

- Types: feat, fix, docs, style, refactor, test, chore, ci, perf, build, revert
- Scopes: backend, frontend, docs, ci, config, keywords, search, dedup, crawler, ocr, rag, writing, chat, pipelines, mcp, i18n, gpu, mineru, prompts, security
- Description in English, imperative mood, lowercase, no period
- Body explains *why* not *what*

## Rules
- Never commit `.env`, `*.db`, `node_modules/`, `__pycache__/`
- Run `make lint` before committing (pre-commit hook handles this)
- No force push to main
- Use feature branches: `feat/*`, `fix/*`, `chore/*`, `docs/*`
- Merge to main via `--no-ff` after CI passes

## Pre-commit
- Install: `make pre-commit-install`
- Hooks: trailing whitespace, ruff lint+format, conventional commit msg
- Branch protection: `no-commit-to-branch` prevents direct main commits

---
> Source: [sylvanding/omelette](https://github.com/sylvanding/omelette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
