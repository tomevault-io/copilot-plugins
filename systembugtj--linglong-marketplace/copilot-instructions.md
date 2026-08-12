## linglong-marketplace

> Claude Code plugin marketplace repo. Plugins live under `plugins/<plugin-name>/`, each with `.claude-plugin/plugin.json` + `skills/<skill-name>/SKILL.md` + `skills/<skill-name>/evals/evals.json`.

# CLAUDE.md

Claude Code plugin marketplace repo. Plugins live under `plugins/<plugin-name>/`, each with `.claude-plugin/plugin.json` + `skills/<skill-name>/SKILL.md` + `skills/<skill-name>/evals/evals.json`.

## Skill authoring rule

Skills ship to arbitrary installers' repos — never bake in this-repo-only specifics (product names, package scopes like `@myorg/ui`, hardcoded paths like `apps/desktop`, origin-project bug reports). Use placeholders (`<app>`, `@scope/pkg`) and tell Claude to inspect the actual repo instead of assuming layout.

When editing a SKILL.md, check its sibling `evals/evals.json` too — eval prompts/expected_output can carry the same repo-specific leak and go stale independently.

## Env quirks

`rtk` (user's global token-saving proxy) intercepts `find` and rejects compound predicates (`-not`, `-exec`): `rtk: rtk find does not support compound predicates or actions`. Use `/usr/bin/find` directly when you need those.

---
> Source: [systembugtj/linglong-marketplace](https://github.com/systembugtj/linglong-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
