## 040-quality-tools

> When PHP code changes:


# Quality tools before commit (short)

When PHP code changes:
- run Pint
- run PHPStan
- run php artisan test

When Markdown changes:
- run markdownlint fix/check (project config)

When security-sensitive changes:
- run gitleaks (checks staged files via pre-commit hook)
- run composer audit

Note: GitLeaks is configured with useDefault=true to detect GitHub tokens and other secrets.
The pre-commit hook runs GitLeaks automatically before each commit.

Do not repeat long command blocks in every response. Mention only what is relevant to the current change.

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
