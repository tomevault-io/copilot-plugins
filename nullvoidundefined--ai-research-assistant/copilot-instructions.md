## ai-research-assistant

> Save articles, papers, bookmarks, notes. RAG pipeline indexes everything. Chat with your knowledge base with streaming citations. Organize into collections. Share via public links. This is the compound app — every AI pattern from apps 1-6 shows up here.

# App 7: AI Research Assistant

Save articles, papers, bookmarks, notes. RAG pipeline indexes everything. Chat with your knowledge base with streaming citations. Organize into collections. Share via public links. This is the compound app — every AI pattern from apps 1-6 shows up here.

## Key AI pattern

Compound AI: structured extraction (app 1) + streaming (app 2) + background processing (app 3) + RAG (app 4) + conversational memory (app 5) + human-in-the-loop citations (app 6).

## Stack

- Monorepo: `packages/api`, `packages/worker`, `packages/web`, `packages/common`
- Next.js on Vercel, Express + BullMQ worker on Railway
- PostgreSQL + pgvector on Neon, Redis on Railway
- Cloudflare R2 for PDFs
- Resend for share notifications
- Reuses chunker, SSE, retrieval, and prompt assembly from prior apps

## Spec

Read `FULL_APPLICATION_SPEC.md` for full system design, DB schema, and task breakdown.

## Build order

POC → save one URL, ingest it, ask a question, get streaming answer with citation → then PDF + notes → then collections → then frontend.

## Shared convention files

Read the relevant file in `.claude/bottomlessmargaritas/` **before writing code** in that layer:

- **Backend:** `.claude/bottomlessmargaritas/CLAUDE-BACKEND.md`
- **Frontend:** `.claude/bottomlessmargaritas/CLAUDE-FRONTEND.md`
- **Database:** `.claude/bottomlessmargaritas/CLAUDE-DATABASE.md`
- **Styling:** `.claude/bottomlessmargaritas/CLAUDE-STYLING.md`
- **Deployment:** `.claude/bottomlessmargaritas/CLOUD-DEPLOYMENT.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nullvoidundefined)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nullvoidundefined)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
