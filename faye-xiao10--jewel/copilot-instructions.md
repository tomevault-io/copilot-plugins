## jewel

> A spatial thinking canvas for learning. Tap to create nodes, auto-connect to nearest neighbor, drag to reposition, click to edit. Voice input, streaks, and multi-canvas management. Built with Next.js, D3, Drizzle, and Neon Postgres.

# CLAUDE.md

## Project
A spatial thinking canvas for learning. Tap to create nodes, auto-connect to nearest neighbor, drag to reposition, click to edit. Voice input, streaks, and multi-canvas management. Built with Next.js, D3, Drizzle, and Neon Postgres.

## Stack
Next.js App Router · React 19 · TypeScript · Drizzle ORM · Neon Postgres + pgvector · Claude Sonnet 4 + Gemini 2.0 Flash · Gemini embedding-001 · umap-js · D3.js · JOSE/JWT · Tailwind CSS · Framer Motion · Vercel

## Commands
```bash

```

## Conventions
- `@/` → `src/`
- Server components by default; `'use client'` only for interactivity
- All LLM calls use tool use, never prompt-based JSON
- API keys server-side only, never exposed to client
- Drizzle schema: `src/db/schema/`, one file per table
- Pipeline tiers: `src/lib/pipeline/`, one file per tier, max 200 lines per file
- Every external API call in try/catch with meaningful error messages
- Descriptive variable names, no abbreviations except: db, req, res, tx
- All UI follows `STYLE.md` — no ad-hoc colors or spacing

## Branching
- Features on `feature/[name]` → merge to `dev` → never direct to `main`
- Always run `git branch` before starting. If on main/dev, create feature branch first.

## Session Protocol

**Start of session:**
1. Read `BUILT.md` to understand current state
2. Read `docs/features/[name].md` for the feature being worked on — use `## Key Files` to load only the files relevant to this feature for context
3. Only read additional docs if needed:
   - Writing queries or schema changes → read `DATA_MODEL.md`
   - Unclear on pipeline/API connections → read `ARCHITECTURE.md`
   - Building UI → read `STYLE.md`
   - Don't read docs you don't need

**During session:**
- Run `/cost` periodically to monitor context usage

**End of session** (triggers: "wrap up", "done", "update docs", "end of feature"):
1. Output a summary of all changes made this session — files touched, what changed, why — and wait for approval before proceeding
2. `pnpm lint && pnpm tsc --noEmit` — fix all errors
3. `find src -type f | sort` — get current file tree
4. Update `BUILT.md` file tree only — reflects current state of `src/` and `docs`
5. Update `DATA_MODEL.md` only if schema changed
6. For each feature file touched this session (`docs/features/[name].md`):
   - Add a new dated entry under `## Built` describing what changed
   - Update the `## Key Files` section — add any new files, annotate changed ones
7. `git add -A` then ONE commit with everything — feature files + docs together. Format: "feat/fix/docs/refactor: description" + bullet points per file.

## Docs (load only what's needed)
| File | Load when |
|------|-----------|
| `BUILT.md` | Start of every session (always) |
| `ARCHITECTURE.md` | Unclear how a feature connects to pipeline or D3 |
| `DATA_MODEL.md` | Writing queries, migrations, or schema changes |
| `STYLE.md` | Building any UI component |
| `docs/features/[name].md` | Working on a specific feature — load only the relevant file |

## Environment Variables
Never log, echo, or commit values. See `.env.local.example` for full list.
Core: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `DATABASE_URL`, `SESSION_SECRET`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `NEXT_PUBLIC_URL`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faye-xiao10)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/faye-xiao10)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
