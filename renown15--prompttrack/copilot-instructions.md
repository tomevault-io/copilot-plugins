## prompttrack

> Always maintain `/Users/marklewis/dev/promptTrack/TASKS.md` throughout every task:

# PromptTrack — Claude Code Rules

## Task list — CRITICAL

Always maintain `/Users/marklewis/dev/promptTrack/TASKS.md` throughout every task:

1. **Before starting** any multi-step work: write the full task list with `- [ ]` items
2. **After completing each step**: immediately update the file, marking that item `- [x]`
3. **When done**: reset the file to `## Status: idle`

**Never batch updates.** Update TASKS.md after every individual step — not at the end.
This means: write the file, do the step, write the file again. Every time. No exceptions.
The file is visible live in the UI; users can see if you are skipping updates.

## Always run before declaring work complete

```
make check   # lint + typecheck + vite build
```

Unit tests alone are not sufficient. `make check` must pass.

## Architecture

- Strict layering: Route Handler → Service → Repository → Prisma
- No Prisma imports outside `packages/api/src/repositories/`
- Shared types via `z.infer<>` only — never hand-write types that mirror Zod schemas
- API files: `kebab-case.repository.ts`, `kebab-case.service.ts`, `kebab-case.routes.ts`
- Pure services: `const camelCase = {...}` object; framework-dependent: `build[Name]Service(fastify)` factory

## Database migrations — CRITICAL

- **Never edit files inside `prisma/migrations/`** after they have been applied
- Only edit `schema.prisma`, then run `make migrate-dev` to generate a new migration file
- Editing applied migration SQL causes Prisma drift and forces a destructive DB reset

## CSS — CRITICAL

- Tailwind only via `@apply` in co-located `.css` files — never inline Tailwind classes in `.tsx`
- BEM class names in `.tsx`
- CSS imports must use `@/` alias — relative imports are banned by ESLint `no-restricted-imports`
  - Correct: `import "@/components/features/chains/ChainNode.css"`
  - Wrong: `import "./ChainNode.css"`

## TypeScript gotchas

- `exactOptionalPropertyTypes: true` — use spread conditionals for optional fields:
  ```ts
  ...(input.description !== undefined && { description: input.description })
  ```
- Same applies to `MutateOptions` — use spread for `onSuccess` etc.

## Naming conventions

- DB: PascalCase models, snake_case tables (`@@map`), camelCase fields, snake_case columns (`@map`)
- Shared schemas: `PascalCase + Schema`, `PascalCase + Input`, `PascalCase + DTO`
- Frontend: `PascalCasePage.tsx`, `usePascalCase.ts`, `camelCaseStore.ts`, `[resource]Api` exports

## Commits

- Max 72 chars header, lowercase subject (e.g. `feat: add chain canvas`)

## Editor

- WYSIWYG editor is TipTap v2 (NOT CodeMirror)
- On save: `editor.storage.markdown.getMarkdown()`
- On load: `editor.commands.setContent(markdownContent)`

## Stack

- pnpm workspaces — use `make` targets, not pnpm directly
- Ports: web=5173, api=3051, postgres=5451
- `packages/shared` — Zod schemas shared by api and web

## Agent Insight — supported report formats

The insight scanner (`discovery.service.ts`, `discovery.per-file.ts`) reads coverage and lint reports from the collection's directory. Both aggregate stats (pipeline tile) and per-file overlays are derived from these files.

### Coverage

| Format                     | File                             | Language |
| -------------------------- | -------------------------------- | -------- |
| Jest / Vitest JSON summary | `coverage/coverage-summary.json` | JS / TS  |
| Python coverage.py JSON    | `coverage.json`                  | Python   |

- Jest/Vitest: detected by presence of `"total"` key; per-file entries are absolute paths with `lines.pct`
- Python: detected by presence of `"totals"` key; per-file entries are under `"files"` with `summary.percent_covered`

### Lint

| Format      | File                  | Language |
| ----------- | --------------------- | -------- |
| ESLint JSON | `.eslint-report.json` | JS / TS  |
| Ruff JSON   | `.ruff-report.json`   | Python   |

- ESLint: array of `{ filePath, errorCount, warningCount }` — one entry per file
- Ruff: array of `{ filename, code, message }` — one entry per violation; errors counted by grouping

### Adding a new format

To add support for another language's coverage or lint tool, update `parseCoverageAggregate` / `parseCoveragePerFile` / `parseLintAggregate` / `parseLintPerFile` in those two service files. Detection is format-based (key sniffing), so no config flags are needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/renown15) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
