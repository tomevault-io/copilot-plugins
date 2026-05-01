## itam-planner

> Apply the repository quality model, automation standards, and release conventions.


@file ../../docs/adr/0006-quality-gates-and-automation.md

# Quality Gates And Automation

- RULE: Use Conventional Commits and CalVer `YY.MM.PATCH`.
- RULE: Derive changelog generation from conventional commit history.
- RULE: Enforce quality with Ruff, basedpyright, ESLint, Prettier, CSpell, Commitlint, markdownlint-cli2, yamllint, ShellCheck, Hadolint, and actionlint.
- RULE: Keep local hooks under `.githooks/`.
- RULE: Prefer shift-left validation over CI-only enforcement.
- RULE: The repository currently runs in single-maintainer mode.
- RULE: `main` is the default working branch and default integration branch for routine development.
- RULE: Short-lived branches and pull requests are optional tools for risky, disruptive, or experimental work, not mandatory workflow steps.
- RULE: Every parser, normalizer, repository, exporter, and applicability resolver must have unit tests.
- RULE: Behavior changes must add or update tests when needed.
- RULE: Use SSH for Git remotes and push operations.
- RULE: GitHub Actions may use `GITHUB_TOKEN` for repository automation when hosted runners cannot use SSH.
- RULE: Stable CI must stay green on `Node.js 24`, while current-line compatibility may run as a non-blocking canary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/humbertoschoenwald) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
