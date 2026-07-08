## kffleague

> - When asked to implement something, START CODING IMMEDIATELY. Do not spend the session exploring and planning unless explicitly asked for a plan.

# QFL — CLAUDE.md

## Core Rules

- When asked to implement something, START CODING IMMEDIATELY. Do not spend the session exploring and planning unless explicitly asked for a plan.
- If you need clarification, ask briefly then proceed with your best understanding.

## Git & Deployment

- This project has 4 separate git repos in one directory: root (kffleague), backend/, qfl-website/, qfl-admin/
- ALWAYS check which repo you're in before git operations (commit, push, status)
- Never push from the wrong root — each repo has its own remote
- Use `/deploy` skill for autonomous deployment pipeline

## Database & Migrations

- Never remove models, database tables, or schema entries without first grep-checking ALL usages across the entire codebase
- Always verify migration dependencies before committing

## UI Development

- When modifying UI components, confirm which specific component/page is meant before starting
- Keep styling changes minimal — no unnecessary shadows, gradients, or !important overrides
- When changing user-facing text, update BOTH translation files (RU and KZ)

---
> Source: [Nurali017/kffleague](https://github.com/Nurali017/kffleague) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
