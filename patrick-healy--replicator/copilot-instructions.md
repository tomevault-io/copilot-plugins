## replicator

> This repository contains an Agent Skill for DCAS compliance auditing.

# Windsurf Rules for replication-compliance

This repository contains an Agent Skill for DCAS compliance auditing.

## Skill Location

The main skill is at `.github/skills/replication-compliance/SKILL.md`

## When to Activate

When user asks about:
- DCAS compliance
- Replication packages
- AEA/journal submission readiness
- Research reproducibility audits

## Workflow

1. Read `.github/skills/replication-compliance/SKILL.md`
2. Follow the audit workflow defined there
3. Use `references/DCAS_RULES.md` for rule details
4. Use `references/LANGUAGE_GUIDES.md` for language-specific checks

## Safety

- Read files but do not run analysis code unless explicitly asked
- Never execute: `git reset --hard`, `git push --force`, `git rebase`
- Flag dangerous commands for manual user execution

## Output

Generate compliance reports following `assets/report_template.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Patrick-Healy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
