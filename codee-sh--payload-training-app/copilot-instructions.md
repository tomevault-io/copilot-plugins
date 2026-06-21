## payload-training-app

> Operational guide for AI agents working in this repository.

# AGENTS.md

Operational guide for AI agents working in this repository.

---

## Before Writing Code

- Write everything in English: code, comments, variable names, documentation.
- Check `.ai/specs/` before coding any non-trivial feature.
- Skills are installed in `.claude/skills/` (Claude Code) and `.agents/skills/` (Codex). `.agents/skills/` is the local source of truth — edit a skill **only** there, then run `ags push-skill` to propagate it to the source repo (it also syncs the `.claude/skills/` copy). See Installing skills.
- Prefer minimal, focused changes. Do not refactor code outside the task scope.
- Run `yarn build` after every implementation to catch type errors.
- Comment and naming conventions are defined in the `code-style` skill. Load it before writing or reviewing TypeScript.

---

## Task Router

Match the task to the table before starting. A single task often maps to multiple rows.

| Task | Action |
|---|---|
| Creating a new collection or extending the schema | Load skill `payload-build-collections` |
| Adding a custom admin view, tab, or field UI | Load skill `payload-build-modules` |
| Building a front-end component or page (`src/components`, `(frontend)`) | Load skill `payload-frontend-build-components` |
| Debugging hooks, queries, access control, transactions | Load skill `payload` |
| Security review, or adding/modifying auth, access control, uploads, CORS/CSRF, headers | Load skill `payload-security`; keep `.ai/audits/security-audit.md` current |
| Writing questions for a client or stakeholder | Load skill `writing-questions` |
| Starting a new spec or reviewing one | Load skill `spec-writing` |
| Any TypeScript code | Load skill `code-style` |

---

## Skills

Skills extend the agent with task-specific guidance, checklists, and reference material.

**Structure:** Each skill lives in its own folder under `.claude/skills/` and `.agents/skills/` with a `SKILL.md` entry point and optional `reference/` files loaded on demand.

**Automatic triggering:** If the task matches a skill's description, load the skill before starting — without waiting to be asked.

**Reference files:** `SKILL.md` specifies which references to load for a given subtask. Do not load all references blindly.

### Installing skills

Skills are managed with [npx skills](https://github.com/vercel-labs/skills). To install from the source repository:

```bash
npx skills add <source-path-or-url> -a claude-code -a codex --copy
```

This installs into `.claude/skills/` (Claude Code) and `.agents/skills/` (Codex). A `skills-lock.json` file tracks the source and version of each installed skill.

### Editing skills

`.agents/skills/` is the local source of truth. Edit a skill **only** there, then push it upstream:

```bash
ags push-skill   # reads .agents/skills, pushes to the source repo, then syncs .claude/skills
```

Do not edit `.claude/skills/` by hand — it is a derived copy that `ags push-skill` overwrites from `.agents/skills/`. Likewise do not hand-edit the source repo; let `ags push-skill` write it.

### Available skills

| Skill | When to load |
|---|---|
| `code-style` | Any TypeScript code |
| `payload` | Debugging Payload: hooks, queries, access control, transactions, security |
| `payload-build-collections` | Creating a new collection or extending the schema |
| `payload-build-modules` | Adding a custom admin view, tab, or field component |
| `payload-frontend-build-components` | Building a front-end React component or page (Next.js + Tailwind) |
| `payload-security` | Security review, or auth/access control/uploads/CORS/CSRF/headers/logging changes |
| `writing-questions` | Writing questions for a client or stakeholder |
| `spec-writing` | Writing or reviewing a feature spec |

### Skill combinations

| When working on | Load |
|---|---|
| Any TypeScript code | `code-style` |
| New collection | `payload-build-collections` + `code-style` |
| New admin view | `payload-build-modules` + `code-style` |
| Front-end component or page | `payload-frontend-build-components` + `code-style` |
| Collection security / access control | `payload-security` + `payload` |

---

## Project Structure

```
src/
├── access/                    # Shared access control functions
├── app/
│   ├── [locale]/(frontend)/   # Client-facing app
│   └── (payload)/             # Payload admin routes and API
├── collections/               # One folder per collection (index.ts + optional hooks.ts, types.ts)
├── components/
│   ├── common/
│   ├── ui/
│   └── workout/
├── data/                      # Static/seed data
├── i18n/                      # next-intl config
├── lib/                       # Utilities and SDK client
├── loaders/                   # Server-side data fetching
├── migrations/                # Payload DB migrations (auto-generated)
├── scripts/                   # One-off CLI scripts
├── types/
├── payload-types.ts           # Auto-generated — do not edit
└── payload.config.ts
.claude/skills/                # Skills for Claude Code
.agents/skills/                # Skills for Codex
.ai/
├── specs/                     # Feature specs (tracked)
└── audits/                    # Security audit status (gitignored — lists unpatched gaps)
```

---

## Key Commands

```bash
yarn dev                    # start dev server
yarn build                  # production build — run after every implementation
yarn payload migrate        # run pending DB migrations
yarn generate:types         # regenerate payload-types.ts
yarn generate:importmap     # regenerate admin import map (after adding custom views)
yarn seed                   # seed demo data
yarn lint                   # ESLint
npx skills add <src> -a claude-code -a codex --copy  # install skills
```

---
> Source: [codee-sh/payload-training-app](https://github.com/codee-sh/payload-training-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
