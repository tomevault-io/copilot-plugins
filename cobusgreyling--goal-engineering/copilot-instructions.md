## goal-engineering

> Canonical public reference for Grok Build CLI `/goal` and `update_goal`. Docs-first; CLIs in `tools/`.

# AGENTS.md — Goal Engineering Reference

## Project Purpose

Canonical public reference for Grok Build CLI `/goal` and `update_goal`. Docs-first; CLIs in `tools/`.

| Path | Contents |
|------|----------|
| `patterns/` | Six goal patterns + `registry.yaml` |
| `starters/` | `minimal-goal`, `tests-green`, `fix-bug` |
| `skills/` | `goal-verifier`, `goal-scoper`, `goal-completion-check` |
| `.grok/skills/` | Grok-native copy of skills (dogfood) |
| `tools/goal-audit` | G0–G3 readiness CLI |
| `tools/goal-init` | Scaffold patterns into projects |
| `tools/goal-cost` | Turn/token estimates |
| `docs/` | API, safety, anti-patterns, multi-goal |
| `GOAL.md` | Active repo objective + done conditions |

## Verification

```bash
npm test
npm run validate
```

Before claiming work done: run tests; use `goal-verifier` mindset on doc accuracy.

## Goal Discipline

- Mirror active `/goal` objectives in `GOAL.md`
- Never `update_goal(completed: true)` without verifier PASS
- One active goal per session
- Log outcomes in `goal-run-log.md`

## Deny List

- npm publish without version bump + release tag
- GitHub org settings or unrelated user repos
- Changing loop-engineering from this repo (cross-link only)

## Code Style

- TypeScript ESM in tools
- Markdown: clear headings, copy-paste `/goal` examples
- Match tone of [loop-engineering](https://github.com/cobusgreyling/loop-engineering)

## Commits

Conventional commits: `docs:`, `feat(goal-audit):`, `feat(goal-init):`, `chore:`.

---
> Source: [cobusgreyling/goal-engineering](https://github.com/cobusgreyling/goal-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
