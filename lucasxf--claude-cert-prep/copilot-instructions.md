## claude-cert-prep

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

CCA-F Exam Simulator — practice exams and study tools for the Claude Certified Architect – Foundations certification.

Dual purpose:
1. **Study tool** — simulate the real exam (60 questions, 120 min, 720/1000 passing score)
2. **Learning exercise** — built with Anthropic SDK and MCP, the exact domains the exam tests

## Stack

- TypeScript (strict mode, ESM), Node.js 20+
- Next.js 15 (App Router) + React 19 + Tailwind CSS
- `@anthropic-ai/sdk` — question generation via Claude API
- `@modelcontextprotocol/sdk` — MCP server for the question bank
- `better-sqlite3` — local SQLite storage (questions + session history)
- `exceljs` — CSV and XLSX export
- `uuid` (v7) — time-ordered UUIDs for all entity IDs
- `vitest` — test runner

## Commands

```bash
npm run dev          # Start Next.js dev server
npm run build        # Production build
npm run start        # Start production server
npm run mcp-server   # Start MCP server standalone
npm run generate     # Generate new questions via Claude API
npm run seed         # Populate the database with seed questions
npm test             # Run all tests
```

To run a single test file: `npx vitest run src/__tests__/scorer.test.ts`

## Architecture

Four components:

**Web App** (`app/`) — Next.js App Router pages, UI in Portuguese:
- `/` — dashboard / start exam
- `/exam/[id]` — active exam session (timer, questions, pause/resume)
- `/exam/[id]/result` — immediate score after submission
- `/exam/[id]/report` — single exam domain breakdown report
- `/history` — consolidated exam history (Histórico)
- `/practice` — focused mini-exams from weak areas

**API Routes** (`app/api/`) — wrap SQLite operations:
- `/api/exams` — create/list exam sessions
- `/api/exams/[id]` — get/update session (pause/resume/submit)
- `/api/exams/[id]/answers` — submit answers
- `/api/reports/[id]` — single exam domain breakdown
- `/api/reports/consolidated` — history table data
- `/api/export/[id]` — export single exam report (CSV/XLSX)
- `/api/export/consolidated` — export history table (CSV/XLSX)

**MCP Server** (`mcp-server/`) — exposes question bank as MCP tools, resources, and prompts via stdio transport. Learning exercise for Domain 2 (Tool Design & MCP Integration).

**Question Generator** (`src/question-generator.ts`) — uses Anthropic SDK with a validation-retry loop (max 3 attempts) to produce structured JSON questions. Learning exercise for Domain 1 (Agentic Architecture).

Static reference data (domains, scenarios, seed questions) in `data/`.
Shared TypeScript types in `src/types.ts`.

## Conventions

- Constructor injection — no global state
- Pure functions where possible
- All shared types in `src/types.ts`
- All entity IDs are UUIDs v7 (time-ordered, sortable)
- Errors must include context (never a bare `throw new Error('failed')`)
- Comments only for architectural decisions, not obvious logic
- 4-space indentation
- UI labels and text in Portuguese; code, specs, and comments in English
- Follow Spec-Driven Development — see `docs/specs/README.md`

## Spec-Driven Development

Feature work follows the SDD lifecycle: Draft → Approved → In Progress → Implemented.

Specs live in `docs/specs/features/`. Use the template at `docs/specs/template.md`.

## Environment

The app runs without any environment variables. `.env.local` is only needed for `npm run generate` (question generation via Claude API).

```bash
# .env.local — only needed for `npm run generate`
ANTHROPIC_API_KEY=sk-ant-...
```

## Branch Strategy

- `main` — production-ready, reached via PRs only
- `develop` — integration branch for feature work

## Security & Correctness Directives

These rules emerged from security and correctness review and must be followed for all future changes.

### Never expose answer keys to the client before exam submission

API routes that return question data to the client during an active exam session **must strip** `correct_answer`, `explanation`, and `wrong_explanations` before responding. Only `get_question` (MCP) and post-submission result/report endpoints may return full question data.

```ts
// Correct pattern in exam creation response:
const safeQuestions = examQuestions.map(
    ({ correct_answer: _ca, explanation: _ex, wrong_explanations: _we, ...q }) => q,
)
```

### Never interpolate non-parameterized values into SQL strings

All values that flow into SQL queries — including numeric limits — must use bound parameters (`?`). String interpolation is forbidden even for seemingly safe values like integers.

```ts
// Wrong:
const limit = filter.limit ? `LIMIT ${filter.limit}` : ''
// Correct:
if (filter.limit) { params.push(filter.limit); limitClause = 'LIMIT ?' }
```

### Always score against the full question set; treat unanswered as incorrect

When scoring an exam, build `AnswerRecord` entries for **every question** in the session, not just answered ones. Filtering out unanswered questions inflates scores and misaligns domain attribution when array indices are reused. Use Maps keyed by `question_id`:

```ts
const questionDomainById = new Map(result.questions.map(q => [q.id, q.domain]))
const answerByQuestionId = new Map(result.answers.map(a => [a.question_id, a]))
const answerRecords = result.questions.map(q => ({
    domain: questionDomainById.get(q.id)!,
    is_correct: answerByQuestionId.get(q.id)?.is_correct === true,
}))
```

### Practice mode requires explicit domain selection — never fall through to exam mode

If `mode === 'practice'` and `domains` is absent or empty, return HTTP 422. Never silently fall through to full-exam question selection while still creating a `practice` session.

### Keep MCP tool schemas in sync with handler implementations

If a tool description mentions a capability (e.g., `exclude_ids`), the `inputSchema` must declare it and the handler must implement it. Either implement the feature or remove the claim from the description — misleading MCP clients breaks integrations silently.

### Fire-and-forget API calls must not use `await`

If a fetch call is intentionally non-blocking (fire-and-forget), do not use `await`. Attach a `.catch()` handler to avoid unhandled promise rejections:

```ts
fetch(url, options).catch(console.error)
```

### Practice sessions must be labeled as such in exports

When generating CSV/XLSX exports, check `session.mode` and output `'Prática'` for practice sessions instead of `'—'`. The export status must match what the history UI displays.

### Seed idempotency requires a UNIQUE constraint in the schema

Catching `SQLITE_CONSTRAINT_UNIQUE` in seed scripts only works if the relevant column has an actual unique index in `schema.sql`. A fresh `uuidv7()` primary key is never a duplicate — dedup logic must target a content column (e.g., `stem`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucasxf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
