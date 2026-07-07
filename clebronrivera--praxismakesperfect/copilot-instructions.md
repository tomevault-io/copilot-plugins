## praxismakesperfect

> This is the primary operating manual for AI agents working in this repo.

# AGENTS.md — Operating Manual for AI Agents

This is the primary operating manual for AI agents working in this repo.
Read this first, then consult the related docs listed below when deeper
context is needed.

---

## Project Summary

Praxis Makes Perfect is a study app for the Praxis 5403 (School Psychology) exam.
It uses React 18 + TypeScript + Vite + Tailwind on the frontend, Supabase (Postgres)
for auth and data, Anthropic Claude for AI features (study plans, tutoring), and
Netlify for hosting and serverless functions. The question bank has 1,000+ questions
across 45 skills in 4 Praxis domains.

---

## Key Paths

| Path | What it is |
|---|---|
| `public/mockup-user-flow.html` | Visual mockup source of truth — check before implementing UI |
| `.claude/plans/` | Active plans (local only, gitignored) |
| `.claude/plans/archive/` | Completed plans (local only, gitignored) |
| `docs/` | Permanent documentation |
| `docs/decisions/` | Architecture Decision Records (ADRs) |
| `scratch/` | Temporary workspace — gitignored contents, never committed |
| `src/data/questions.json` | Question bank (1,000+ questions) |
| `src/data/question-skill-map.json` | Skill-to-question mapping |
| `src/brain/` | Assessment logic, distractor patterns, question generation |
| `src/utils/progressTaxonomy.ts` | Canonical 4-domain Praxis 5403 model |
| `src/utils/skillProficiency.ts` | Proficiency thresholds (Emerging / Approaching / Demonstrating) |
| `api/` | Netlify serverless functions |
| `supabase/migrations/` | Database migrations — applied via `supabase db push` |

---

## Commands

| Command | What it does |
|---|---|
| `npm install` | Install dependencies |
| `npm run dev` | Start Vite dev server (port 5173) — no serverless functions |
| `npm run dev:netlify` | Start full dev environment via Netlify CLI (port 8888) — includes functions |
| `npm run build` | Typecheck then build for production |
| `npm run scan:types` | TypeScript type check (`tsc --noEmit`) |
| `npm run lint` | ESLint with `--max-warnings 0` |
| `npm run test` | Run Vitest test suite |
| `npm run check` | Full quality gate: typecheck + lint + test |

**Use `npm run dev:netlify` (not `npm run dev`)** if you need study plan generation,
AI tutor, leaderboard, or any other serverless function to work.

---

## Definition of Done

Before marking any task complete:

1. `npm run scan:types` passes
2. Run `npm run test` when the change affects tested logic
3. Run `npm run lint` and do not introduce new lint violations in touched files
4. Manual smoke test of affected functionality
5. No files left in `/scratch/` that should be committed
6. If a plan in `.claude/plans/` is complete, move it to `.claude/plans/archive/`
7. If the change affects user-facing behavior documented in `docs/HOW_THE_APP_WORKS.md`,
   update that file in the same commit

---

## Folder Rules

| Location | Use for | Committed to git? |
|---|---|---|
| `scratch/` | Temporary analysis, audits, exploration, throwaway files | No (gitignored) |
| `docs/` | Permanent documentation, handoffs, reference material | Yes |
| `docs/decisions/` | Architecture Decision Records (ADRs) | Yes |
| `.claude/plans/` | Active plans for in-progress work | No (gitignored — local agent workspace) |
| `.claude/plans/archive/` | Completed plans | No (gitignored — local agent workspace) |

**Rules:**
- Never create loose `.md` files at the repo root. Use `docs/` for permanent docs,
  `scratch/` for throwaway work.
- `.claude/plans/` and `.claude/plans/archive/` are local-only conventions.
  The entire `.claude/` directory is gitignored, so these paths exist on the
  developer's machine but are not tracked or enforced by git.

---

## Do Not Touch Without Asking

These files are high-risk. Do not modify them without explicit approval from the user.
If a task requires changes to any of these, state the need, explain the impact,
and wait for a go-ahead.

- `src/brain/skill-map.ts` — skill definitions and domain assignments
- `src/data/questions.json` — the question bank
- `src/data/question-skill-map.json` — question-to-skill mapping
- `src/utils/progressTaxonomy.ts` — canonical 4-domain Praxis model
- `supabase/migrations/` — any migration file (create new ones, don't edit existing)
- Legacy Firebase / Firestore config or rules files, if present
- `.env`, `.env.local`, `.env.test` — environment files with credentials

---

## Visual-First Rule

When implementing or redesigning UI features:

1. For significant UI changes, check `public/mockup-user-flow.html` first.
   If the target screen is not yet implemented there, create or update the mockup
   before changing React components unless the user explicitly directs otherwise.
2. Unimplemented mockup screens take priority over backend work unless the user
   explicitly says otherwise.
3. Get visual approval before writing React code.

See `CLAUDE.md` — "UI Redesign Workflow — Mandatory Mockup-First Rule" for the
full rationale and constraints.

---

## Plan Before Code

Before writing code on any non-trivial change:

1. State your assumptions
2. Propose a plan
3. Wait for approval

**Approval is required** for anything that:
- Touches more than 3 files
- Modifies any file on the "Do Not Touch" list
- Changes database schema or migrations
- Alters assessment logic, scoring, or proficiency thresholds
- Adds or removes dependencies

---

## Related Docs

| Document | Purpose |
|---|---|
| `CLAUDE.md` | Deep implementation context — auth, migrations, thresholds, architecture, credentials |
| `docs/WORKFLOW_GROUNDING.md` | Durable implementation and reporting rules |
| `docs/ISSUE_LEDGER.md` | Active discovered issues, bugs, and reporting mismatches |
| `docs/HOW_THE_APP_WORKS.md` | Canonical plain-language product description (must stay current) |
| `docs/DOCS_SYSTEM.md` | Map of what each doc is and when to update it |
| `.cursor/rules/agent_protocols.mdc` | Cursor-specific agent protocols |
| `.cursor/rules/domain_rules.mdc` | Domain-specific data constraints (Praxis 5403 model) |

---
> Source: [clebronrivera/PraxisMakesPerfect](https://github.com/clebronrivera/PraxisMakesPerfect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
