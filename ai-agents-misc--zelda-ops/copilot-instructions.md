## zelda-ops

> > Satellite plugin for zelda — provides skills for environment setup,

# Zelda Ops — Operational & Authoring Skills

> Satellite plugin for zelda — provides skills for environment setup,
> plugin maintenance, security scanning, defriction, and skill development.

## Team Rules

**Follow `TEAM-RULES.md` in the zelda repo.** All team-wide rules apply here.

## Scope

This plugin contains standalone operational skills extracted from zelda core.
These skills have zero inbound dependency edges — nothing in zelda chains into
them. They are user entry points only.

Skill categories:

1. **Environment & config** — setup, backup, diagnose environment
2. **Plugin authoring** — create-skill, create-hook, uninstall-plugin
3. **Security & maintenance** — security-scan, folder-cleanup, upstream-watch
4. **Defriction & learning** — defriction, pr-learnings, guess-nicks-pick
5. **Documentation** — audit-claude-md, claude-md-diet, prompt-refactor, using-zelda
6. **Execution modes** — compete, schedule, repo-watch

**This plugin does NOT contain** core workflow skills (PRD, plan, execute,
debug, simplify, review-pr, etc.) — those remain in zelda.

## Architecture

- Skills live in `skills/*/SKILL.md`
- This is a satellite of the zelda plugin family — not a standalone plugin
- Invocation prefix is `/zelda-ops:`, not `/zelda:`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AI-Agents-Misc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
