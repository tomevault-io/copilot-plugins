## agent-skills-pack

> description: Project conventions for Agent Skills Pack

---
description: Project conventions for Agent Skills Pack
alwaysApply: true
---

# Agent Skills Pack Conventions

## Prompt the User in Chat

When the user is working in this repo, **proactively prompt them in chat** to run validation and certification when relevant:

- **After editing SKILL.md or skill-related files** — Remind: "You've edited skill files. Run `npm run validate` before committing, or I can run it for you."
- **When the user mentions commit, push, or "ready to ship"** — Remind: "Before committing, run `npm run validate` and `npm run lint:md`. For certification score: `npm run score`."
- **When the user asks about project health or status** — Offer: "I can run validation and certification score for you. Say 'run validation' or 'run score'."
- **When starting a session involving skills** — If context suggests skill work, offer: "Want me to run validation or certification score?"

Do not be repetitive; prompt once per relevant context. If the user declines or has already run checks, do not prompt again in the same thread.

## Before Committing

- Run `npm run validate` (skills manifest and structure)
- Run `npm run lint:md` (markdown lint)
- Pre-commit runs validate; pre-push runs validate + lint:md

## Adding a New Skill

1. Create skill folder with SKILL.md, examples.md, prompt-template.md, reference.md
2. Update `skills-manifest.json` with the new skill
3. Run `npm run validate` to verify

## File Structure

Each skill: `SKILL.md`, `examples.md`, `prompt-template.md`, `reference.md`

## Engineering Intelligence

This pack aligns with DORA, DORA AI Capabilities, DX, and HEART. See [docs/engineering-intelligence-framework.md](docs/engineering-intelligence-framework.md). Rules enforce small batches, safe deployments, rollback readiness, and secure AI usage.

## Bypass Hooks

Use `git push --no-verify` only when necessary; CI still enforces validation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LongTheta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
