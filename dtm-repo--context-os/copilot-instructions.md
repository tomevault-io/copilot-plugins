## context-os

> This folder is the user's portable Context OS — a cross-project AI context layer. It is intended for Codex-style coding agents and other autonomous AI agents that use repository-local instruction files.

# Agent Instructions for Context OS

This folder is the user's portable Context OS — a cross-project AI context layer. It is intended for Codex-style coding agents and other autonomous AI agents that use repository-local instruction files.

> **First-time setup:** If the `YOU_*.md` files are still mostly empty/template text, read `SETUP.md` and run the setup flow — pre-fill from what you already know about the user, then confirm. Don't make them fill in blank files by hand.

## Mission

Use these files to maintain smooth continuity across the user's projects, AI platforms, and coding environments.

Do not treat this as a static profile. Treat it as a source-of-truth context system that can be improved through careful, explicit updates.

## Read Order

Before beginning substantial work, read (consult `MANIFEST.md` for how much to load):

1. `YOU_CONTEXT.md`
2. `YOU_PROJECTS.md`
3. `YOU_TECH_STACK.md`
4. Relevant project folder under `projects/`
5. Current user task or repository-specific instructions

Current user instructions override these files. Project-specific instructions override global context.

## Working Style

Default style (override with the user's own notes in `YOU_CONTEXT.md`):

- exact commands, explicit paths, patch-level guidance
- practical explanations and clear verification steps
- durable context updates when warranted

Avoid: vague architecture advice, motivational filler, generic productivity language, unsupported claims, and invented biographical or project facts.

## Technical Work

When modifying code or guiding technical work:

- Make small, understandable changes.
- Explain which files changed and why.
- Include commands to run and the expected output / success condition.
- If tests cannot be run, say why and give the verification path.
- Honor the user's stack preferences as recorded in `YOU_TECH_STACK.md`.

## Context Maintenance

At the end of meaningful work, identify whether the session created durable updates for `YOU_CONTEXT.md`, `YOU_PROJECTS.md`, `YOU_TECH_STACK.md`, `YOU_CAREER.md`, `YOU_CREATIVE.md`, the private overlays, or a project-specific `PROJECT_CONTEXT.md` / `CLAUDE.md` / `AGENTS.md`.

Return proposed edits clearly. Do not bury context changes inside unrelated code changes.

## Canonical Merge Principle

No model's previous version is automatically primary.

When multiple context versions conflict, synthesize toward the clearest, most accurate, most useful canonical version. You may add, remove, consolidate, reframe, or de-emphasize material. The desired outcome should be substantially independent of the order in which different AI systems contributed.

## Current-State Truth Model

Context OS is meant to become the maintained current state, not a permanent set of competing observations.

During onboarding or cross-agent reconciliation, treat each source as partial evidence. Use provenance while reconciling uncertainty. Once the user confirms a fact or enough evidence supports it, promote it into current-state language. In normal operation, the current files are the working truth; the user's corrections override them. New evidence should update the files rather than force future agents to re-litigate settled facts.

## If Reviewing After Another Agent

The user may hand this folder to you after another agent's pass.

If so, do not rubber-stamp the prior edits and do not treat that structure as authoritative. Treat it as one contribution to review independently:

- check whether `AGENTS.md` is sufficient for your coding sessions
- check whether `CLAUDE.md` contains assumptions that should be generalized elsewhere
- identify project context that belongs in project folders rather than global files
- preserve useful guidance while improving platform-neutral continuity
- return a patch or replacement sections with reasoning

## Privacy

Do not add sensitive family, employment, salary, legal, or private project details to broadly shared files without the user's confirmation. Use `YOU_PRIVATE.md` / `YOU_CAREER_PRIVATE.md` (git-ignored) for sensitive context, and only when the user wants it stored.

---
> Source: [DTM-repo/Context-OS](https://github.com/DTM-repo/Context-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
