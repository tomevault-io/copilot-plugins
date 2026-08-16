## production-rag-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Current state

**The repository is empty.** [instructions.txt](instructions.txt) is the full project specification and is the authoritative source for anything not covered here. No code, no git history, no package manifests exist yet.

Work begins at **Phase 1 (Foundation)**. Do not implement document ingestion or anything downstream of it until Phase 1 is complete and validated.

Because nothing is scaffolded, the commands below describe the contract each app must satisfy once created — verify a manifest actually defines a script before relying on it.

## Project

`production-rag-platform` — a document question-answering platform. The end-to-end flow: upload a document → extract and normalize text → chunk → embed → store chunks + vectors in PostgreSQL/pgvector → user asks a question → retrieve relevant chunks → generate a grounded answer with passage-level citations → record retrieval, latency, token usage, and evaluation metadata.

Independent portfolio implementation. No proprietary code, confidential data, or employer-specific architecture.

## Stack

- **Frontend** (`apps/web`): Next.js, React, TypeScript, Tailwind, React Hook Form, Zod, SSE for streaming
- **Backend** (`apps/api`): Python 3.12, FastAPI, Pydantic, SQLAlchemy, Alembic, PostgreSQL + pgvector, Redis, OpenAI-compatible model interface
- **Infra**: Docker Compose, GitHub Actions, structured logging, environment-based config
- **Tests**: Pytest (backend), Vitest + React Testing Library (frontend)

Deviate from this stack only for a clear technical reason, and record it as an ADR.

## Commands

Backend (`apps/api`):

```bash
ruff check .
mypy .
pytest
pytest path/to/test_file.py::test_name   # single test
alembic upgrade head
```

Frontend (`apps/web`):

```bash
npm run lint
npm run typecheck
npm run test
npm run test -- path/to/file.test.tsx    # single test file
npm run build
```

Infrastructure:

```bash
docker compose config
docker compose up --build
```

Run the checks relevant to a change **before** proposing a commit, and report the actual results. Do not propose a commit while relevant validation is failing — the only exception is an explicitly-labeled failing test in a TDD sequence.

## Repository structure

```text
apps/web/            Next.js frontend
apps/api/            FastAPI backend
packages/retrieval/  Retrieval logic
packages/evaluation/ Eval metrics and scoring
packages/shared/     Cross-cutting types/utilities
docs/                architecture.md, api.md, security.md, evaluation.md, decisions/
evals/               datasets/ (JSONL), reports/, runners/
scripts/
tests/
.github/workflows/, .github/ISSUE_TEMPLATE/
```

Keep it pragmatic — do not create a package until something real lives in it.

## Architecture constraints

These are the decisions that shape the code and are expensive to retrofit:

- **Ingestion and query are separate paths.** Ingestion (extract → chunk → embed → store) must be idempotent and re-runnable; re-ingesting the same document must not duplicate chunks or vectors. Duplicate detection is by content hash.
- **Providers are abstractions, not direct SDK calls.** Both the embedding provider and the model provider sit behind an interface, with retry, timeout, and error handling in the abstraction rather than at call sites.
- **Grounded mode is enforced, not suggested.** When configured for document-grounded answering, the model must not answer from general knowledge. Insufficient retrieved context must produce a refusal. Every answer carries citation identifiers tied to retrieved passages.
- **Model responses are structured and validated** against a schema before reaching the client.
- **Retrieved document content is untrusted input** and must be kept structurally separate from system instructions in the prompt. Prompt-injection limitations belong in `docs/security.md`.
- **Retrieval starts vector-only.** Add hybrid/keyword ranking only after the vector baseline is tested.
- **Never log document contents.** Log retrieval diagnostics, latency, and token counts — not the text.

## Data model

UUID primary keys, `created_at`/`updated_at` on every table, explicit indexes and foreign keys. Core tables: document, chunk, ingestion job, chat session, message, retrieval event, evaluation run. All schema changes go through Alembic.

## Phases

Build one coherent capability at a time; each should leave the repo in a valid state.

1. **Foundation** — monorepo, Next.js + FastAPI init, tooling, Docker Compose with PostgreSQL and Redis, health/readiness endpoints, README ← *current*
2. **Persistence** — DB config, models, Alembic
3. **Ingestion** — upload, type/size validation, safe filenames, PDF/text/markdown extraction, token-aware chunking with configurable size and overlap, content-hash dedupe, status tracking
4. **Embeddings and vector storage** — provider abstraction, batching, retries, pgvector storage, embedding model metadata, idempotency
5. **Retrieval** — similarity search, top-k, metadata filters, similarity threshold, dedupe, diagnostics
6. **Grounded generation** — prompt construction, context limits, structured responses, citations, refusals, streaming, token/latency metrics
7. **Frontend** — document list, upload + progress, ingestion status, streaming chat, citation panel, source preview, error/retry states, accessible responsive layout
8. **Evaluation** — JSONL datasets, retrieval recall, citation correctness, groundedness, relevance, latency, tokens, estimated cost, baseline comparison, regression thresholds, reproducible reports
9. **Reliability** — Redis caching, request IDs, structured logs, rate limiting, timeouts, retries, graceful provider degradation, secret redaction
10. **CI and release** — full lint/typecheck/test/build matrix, Docker + migration validation, dependency security checks, PR/issue templates, CONTRIBUTING, CHANGELOG, semver

## Commits

Conventional Commits: `<type>(<scope>): <concise description>`.

Types: `feat` `fix` `test` `docs` `refactor` `perf` `chore` `ci` `build` `infra`. Scopes in use: `repo` `web` `api` `db` `ingestion` `embeddings` `retrieval` `chat` `evals` `cache` `observability` `resilience` `metrics` `tooling` `local` `readme` `architecture` `contributing` `changelog` `release`.

One coherent change per commit, no unrelated formatting churn, tests and docs updated alongside behavior. A normal feature is one to four commits — do not pad to hit a number.

**Do not run `git commit` unless explicitly instructed.** Recommend a message instead.

**Never modify author or committer dates**, and never fabricate development history. If pre-existing owner code is incorporated, preserve its authorship, timestamps, and license, and document its origin.

## Reporting format

After each completed step:

```text
Completed:
- Summary of implemented behavior

Files changed:
- Relevant files

Validation:
- Commands run
- Results

Recommended commit:
- type(scope): concise description

Next step:
- One logical follow-up task
```

## Working rules

- Inspect existing code before changing it; avoid touching unrelated files.
- Implement the smallest coherent production-quality version. Add abstraction only when a visible requirement needs it.
- Clearly mark mocked external services and sample data as such.
- Never claim unsupported performance, reliability, or scale figures; no fabricated users, stars, testimonials, or metrics.
- Substantial work follows the PR flow: issue → branch from `main` (`feat/…`, `fix/…`, `test/…`, `docs/…`) → tests → docs → validation → PR describing the trade-offs → merge on green CI.
- Meaningful architectural decisions get an ADR in `docs/decisions/` with Context, Decision, Alternatives, Consequences, Status.

---
> Source: [tzheng403/production-rag-platform](https://github.com/tzheng403/production-rag-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
