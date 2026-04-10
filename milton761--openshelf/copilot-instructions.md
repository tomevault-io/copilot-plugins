## openshelf

> This repository is designed to be maintained by AI coding agents.

# OpenShelf Agent Guide

This repository is designed to be maintained by AI coding agents.

## Project intent
- Build a local-only EPUB reader in React.
- No backend, no cloud sync, no external book uploads.
- Keep UX simple and focused on reading.

## Working agreements
- Make minimal, focused changes.
- Do not introduce frameworks beyond React + Vite unless requested.
- Keep dependencies small and justified.
- Preserve local-first behavior.

## Standard workflow
1. Read `README.md` and relevant files in `src/`.
2. Create/update a short plan before complex changes.
3. Implement changes with small diffs.
4. Run `npm run build` before handoff.
5. Update docs when behavior or scripts change.

## Common commands
- `npm install`
- `npm run dev`
- `npm run build`
- `npm run lint`

## Invoking skills from chat
- Skills are prompt-triggered: ask for the skill by name and desired outcome.
- Example: "Use `openshelf-bugfix` to diagnose blank EPUB rendering and patch it with minimal changes."
- Example: "Use `openshelf-git-ops` to stash unrelated edits, commit the current feature, and push safely."

## Available skills
- `.agents/skills/openshelf-feature/SKILL.md`
- `.agents/skills/openshelf-bugfix/SKILL.md`
- `.agents/skills/openshelf-docs-release/SKILL.md`
- `.agents/skills/openshelf-docs-maintainer/SKILL.md`
- `.agents/skills/openshelf-git-ops/SKILL.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Milton761)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Milton761)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
