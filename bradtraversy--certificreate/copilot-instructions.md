## certificreate

> Instructions for AI coding agents working in this project. This is the cross-tool

# AGENTS.md

Instructions for AI coding agents working in this project. This is the cross-tool
entry point: Codex, Cursor, GitHub Copilot, Gemini CLI, Aider, Zed, Windsurf, and
others read `AGENTS.md`. Claude Code reads `CLAUDE.md`, which imports this file, so
there is a single source of truth.

## What this is

**Certificreate** - a Next.js 16 (App Router) + TypeScript + Tailwind v4 app.

> TODO: one-line product description. Fill in `blueprint/project-plan.md`, then
> run `/overview`; update this line to match.

This project is built with the **AI Coding Blueprint**, a workflow layer, not an
app skeleton. To start a new project, scaffold the app first in an empty folder
(create-next-app, Vite, etc.), then overlay these files on top. Never run a
framework scaffolder inside a directory that already holds the blueprint files
(`AGENTS.md`, `CLAUDE.md`, `.agents/`, `.claude/`, `blueprint/`); it fails
because the directory isn't empty.

New here? `blueprint/README.md` explains the whole workflow.

## Read these for full context

- `blueprint/context/project-overview.md` - the project's source of truth
- `blueprint/context/coding-standards.md` - conventions to follow
- `blueprint/context/ai-interaction.md` - how to work with the user on this project
- `blueprint/context/current-feature.md` - the one feature or fix being built right now

## Workflow

Build one feature or fix at a time, behind review gates. Each step's instructions
are plain markdown skills any capable agent can read and follow. The workflow is
exposed through tool-specific adapters:

- Claude Code: `.claude/skills/<skill>/SKILL.md`

This project is Claude Code-only: `.claude/` is the only adapter installed (no
`.agents/`). Keep `AGENTS.md` because `CLAUDE.md` imports it, and it remains the
entry point for any other tool that reads `AGENTS.md`.

Core skills:

- `onboard` - tune commands, standards, visibility, ignore rules, and tool adapters after overlaying the Blueprint onto a freshly scaffolded or early project
- `doctor` - read-only Blueprint health check for setup, adapters, plans, overview freshness, and workflow drift
- `adopt` - bootstrap the Blueprint into an existing brownfield app with shipped features
- `overview` - distill the two planning docs into `blueprint/context/project-overview.md`
- `brief` - read-only briefing on an upcoming build-plan feature (scope, dependencies, size) before you spec it
- `feature` - turn a build-plan item into a spec in `blueprint/context/current-feature.md`
- `fix` - document an ad-hoc bug or change into `blueprint/context/current-feature.md`
- `tests` - add or normalize unit testing and turn on the test gate
- `implement` - build the current spec one small, reviewed step at a time
- `check` - prove the current spec against the running app
- `try` - read-only manual review guide: where to go, what to click, what to expect
- `audit` - read-only code quality review for duplication, dead code, standards drift, and maintainability risks
- `complete` - log it to `blueprint/history/features/` or `blueprint/history/fixes/`, then merge
- `prototype` - optional, pre-build static mockups to lock the look
- `status` - read-only progress summary, workflow drift warning, and suggested next action

In Claude Code, use the slash commands (`/onboard`, `/overview`, `/feature`, and
so on) or ask naturally, such as "run the overview." In tools without native
skills, follow the matching `SKILL.md` manually. The conventions in
`blueprint/context/` apply however a step is invoked.

Optional explicit-only skill: `autopilot` can run one bounded spec/build/check
pass when directly invoked. It may create checkpoint commits on the feature or
fix branch after passing steps. It stops before `/complete`, merge, push, deploy,
or destructive actions.

## Commands

Package manager: npm (package-lock.json).

- Dev server: `npm run dev` (http://localhost:3000)
- Build: `npm run build`
- Production server: `npm run start`
- Lint: `npm run lint`
- Test: `npm test` (Vitest, single run)
- Test watch: `npm run test:watch`

A `test` command is declared, so the testing gate in
`blueprint/context/coding-standards.md` applies: build steps that add in-scope
logic must ship a passing test in the same diff, and the suite must be green
before a step is approved, before any checkpoint commit, and before `/complete`
merges.

---
> Source: [bradtraversy/certificreate](https://github.com/bradtraversy/certificreate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
