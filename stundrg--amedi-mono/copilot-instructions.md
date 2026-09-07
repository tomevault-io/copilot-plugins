## amedi-mono

> This repository is a monorepo for the AmedI healthcare assistant project:

# AmedI Agent Instructions

## Scope

This repository is a monorepo for the AmedI healthcare assistant project:

- `apps/backend`: FastAPI backend using Python 3.12, PDM, PostgreSQL, FAISS, and LLM/RAG services.
- `apps/frontend/my-app`: Next.js frontend using pnpm, React, TypeScript, Tailwind CSS, and Radix UI.
- `qa`: QA plans, manual validation checklists, release criteria, and test evidence.
- `tests`: Cross-app and end-to-end test assets that do not naturally belong to one app.

## Default workflow

- Respond to the user in Korean unless the user explicitly asks for another language.
- Prefer small, targeted changes over broad refactors.
- Read only the files needed for the current task.
- Use `rg` for search when possible.
- Do not run full test suites, builds, or long-running commands unless explicitly requested.
- Do not use MCP by default. Add MCP only for a clearly scoped workflow, such as future push-triggered code review.
- Ask before destructive filesystem operations, force pushes, Git history rewrites, or deployment credential changes.

## Review rules

- Favor correctness over style.
- When generating implementation code, include corresponding test code unless the user explicitly asks to skip tests.
- For backend code, inspect error handling, transaction boundaries, idempotency, and retry safety.
- For API changes, check backward compatibility and response schema changes.
- For database changes, verify migration safety and rollback risk.
- For frontend changes, check API contract drift, auth boundaries, loading/error states, and runtime environment assumptions.
- For AI/RAG changes, check prompt/input boundaries, fallback behavior, generated index compatibility, and deterministic test coverage where practical.
- For QA changes, keep acceptance criteria and evidence links close to the scenario being validated.
- Always mention missing tests if behavior changed.

## Token discipline

- Start with narrow file or symbol searches.
- Avoid broad directory listings unless the task is repository-structure work.
- Summarize large outputs instead of copying them.
- Prefer existing root scripts over ad-hoc command chains.
- Split large work into clear phases and stop after each phase for review.

## Documentation rules

- For task-specific document selection, follow `docs/guides/agent-workflow.md`.
- Architecture decisions go in `docs/adr/`.
- Product requirements go in `docs/prd/`.
- System diagrams and ERD live in `docs/architecture/` and should use Mermaid where practical.
- Developer setup and conventions live in `docs/guides/`.
- QA plans and manual validation checklists live in `qa/`.
- Shared test harnesses and cross-app tests live in `tests/`.

## Git rules

- Commit format: `<type>: <description>`.
- Common types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`.
- Do not commit secrets, local `.env` files, virtualenvs, build outputs, or generated caches.
- Only commit when explicitly requested.

---
> Source: [stundrg/amedi_mono](https://github.com/stundrg/amedi_mono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
