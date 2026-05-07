## wiki-source-code

> LLM Wiki Agent — TypeScript source code conventions


# TypeScript Source Code Conventions

## Project Stack

- **Runtime**: Node.js 22.16.0 + TypeScript (ESM)
- **LLM**: Anthropic Claude via `@anthropic-ai/sdk`
- **Graph**: graphology + graphology-communities-louvain + vis.js
- **CLI**: commander
- **Package manager**: pnpm

## Architecture

```
src/
├── shared/        # Shared modules (types, constants, utils)
├── workflows/     # Four core workflows (ingest, query, lint, graph)
└── cli/           # CLI entry points (thin wrappers over workflows)
```

- Business logic lives in `src/workflows/`, CLI parsing lives in `src/cli/`.
- Shared utilities in `src/shared/` — do not duplicate code across workflows.
- All imports use `.js` extension (ESM requirement).
- Model names are centralized in `src/shared/constants.ts`.

## Quality

- Run `pnpm typecheck` before committing.
- Run `pnpm lint` and `pnpm format` to check code standards.
- TypeScript strict mode is enabled — no `any` types unless absolutely necessary.

---
> Source: [regexp-lin/llm-wiki-agent](https://github.com/regexp-lin/llm-wiki-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
