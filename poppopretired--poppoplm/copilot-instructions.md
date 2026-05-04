## poppoplm

> This file is read automatically by AI coding agents. It defines the project stack, directory conventions, tiered context protocol, and the feature-scoping checklist you must follow before building any feature.

# PopPopLM Agent Context Guide

This file is read automatically by AI coding agents. It defines the project stack, directory conventions, tiered context protocol, and the feature-scoping checklist you must follow before building any feature.

---

## Stack — Pinned Versions

| Layer            | Package               | Version                                      |
| ---------------- | --------------------- | -------------------------------------------- |
| Runtime          | Bun                   | latest                                       |
| Web Framework    | Hono                  | 4.12.7                                       |
| Validation       | Zod                   | **4.3.6** (v4 — breaking changes from v3)    |
| Authentication   | @hono/auth-js         | 1.1.1                                        |
| Auth Core        | @auth/core            | 0.34.3                                       |
| AI Framework     | @mastra/core          | **1.13.2** (v1.x — pre-1.0 APIs are invalid) |
| AI CLI           | mastra                | 1.3.12                                       |
| AI Hono Adapter  | @mastra/hono          | 1.2.4                                        |
| AI Memory        | @mastra/memory        | 1.8.2                                        |
| AI Storage       | @mastra/libsql        | 1.7.0                                        |
| AI Logging       | @mastra/loggers       | 1.0.2                                        |
| AI Observability | @mastra/observability | 1.5.0                                        |
| ORM              | drizzle-orm           | 0.45.1                                       |
| ORM Migrations   | drizzle-kit           | 0.31.9                                       |
| ORM+Zod Bridge   | drizzle-zod           | 0.8.3                                        |
| Database Client  | @libsql/client        | 0.17.0                                       |
| AI Provider      | @ai-sdk/openai        | 3.0.47                                       |
| AI Embeddings    | @xenova/transformers  | 2.17.2                                       |
| Ingestion HTML   | cheerio               | 1.2.0                                        |
| Ingestion PDF    | pdf-parse             | 2.4.5                                        |
| Ingestion Media  | youtube-transcript    | 1.3.0                                        |
| CSS Framework    | tailwindcss           | **4.2.1** (v4 — CSS-first, no JS config)     |
| UI Components    | daisyui               | **5.5.19** (v5 — @plugin in CSS)             |
| Client Behavior  | alpinejs              | **3.15.8** (directive syntax is strict)       |
| Alpine Types     | @types/alpinejs       | **3.13.11** (TypeScript typings only)         |
| Icons            | @iconify/tailwind4    | **1.2.3** (CSS-first plugin)                 |
| Icon Set         | @iconify-json/lucide  | 1.2.97                                       |
| Hypermedia       | htmx.org              | **2.x** (v2 — event syntax changed)          |
| Route Validation | @hono/zod-validator   | 0.7.6                                        |
| Testing          | Bun test + happy-dom  | built-in / 20.8.4                            |


`package.json` is the absolute source of truth for versions.

---

## Directory Conventions

```
PopPopLM/
  src/
    index.tsx           ← Hono app entry point
    styles.css          ← Tailwind + DaisyUI + Iconify config (source)
    db/
      index.ts          ← drizzle client singleton
      schema.ts         ← all table definitions
      queries/          ← typed query helpers per domain
    lib/                ← shared helpers and pure logic
    types/              ← shared TS types
    routes/             ← Hono route groups (one file per domain or `<name>/index.tsx`)
    mastra/
      index.ts          ← Mastra instance
      agents/           ← one file per agent
      tools/            ← one file per tool
      workflows/        ← one file per workflow
    views/              ← full-page Hono JSX views
    components/         ← reusable HTMX partial components
  public/
    styles.css          ← compiled CSS output (do not edit)
  drizzle/              ← generated migration files
```

---

## Tiered Context Protocol

Apply context in tiers. Never attach all docs simultaneously — it degrades output quality.

### Tier 1 — Always Active (automatic via skills)

These rules are auto-applied by Antigravity via the skills in `.agents/skills/`:

| Skill               | Scope                                                     |
| ------------------- | --------------------------------------------------------- |
| `project (Standards)` | Every file — version pins, TypeScript, Check-Before-Write |
| `mastra`            | `src/mastra/**` — agent/tool/workflow patterns            |
| `database`          | `src/db/**` — schema, queries, migrations                 |
| `ui`                | `src/views/**`, `src/components/**`, `*.css` — UI layer   |
| `ingestion`         | `src/lib/ingest-source-content.ts`, `src/routes/sources.tsx`, `src/db/queries/sources.ts` — Source parsing |

### Tier 2 — Per Feature (attach manually at session start)

Attach only the docs relevant to the current feature. Keep total context to: 1 rule file + 1–2 source files + 1 doc URL.

| Building...                   | Attach                                                                                           |
| ----------------------------- | ------------------------------------------------------------------------------------------------ |
| AI agents / tools / workflows | `@Mastra Docs` + `src/mastra/index.ts` + `src/routes/` + the specific agent file        |
| Database schema or queries    | `@Drizzle Docs` + `src/db/schema.ts`                                                             |
| Zod validation schemas        | `@Zod v4 Docs` + `src/db/schema.ts`                                                              |
| Hono routes or middleware     | `@Hono Docs` + `src/index.tsx` + relevant route file                                             |
| UI views or components        | `@Tailwind Docs` + `@DaisyUI Docs` + `@Alpine Docs` + `@HTMX Docs` + `@Iconify Tailwind4 Docs` + `src/styles.css` |
| Agent-UI integration          | `@Mastra Docs` + `@Hono Docs` + `src/mastra/index.ts` + relevant route file                      |
| Tests                         | `@Bun Docs` + the source file under test                                                         |
| Authentication & Sessions     | `@Auth.js Docs` + `src/lib/auth.ts` + relevant route/view                                        |

### Tier 3 — On-Demand (debugging only)

When a specific API throws an error or behaves unexpectedly:

- Use **web search or read_url_content** to fetch the live, version-accurate API signature for the offending package.
- Attach the package changelog or migration guide only for that package.

---

## Documentation URLs

| Package             | Primary Docs                                              | Notes                      |
| ------------------- | --------------------------------------------------------- | -------------------------- |
| Mastra              | https://mastra.ai/docs                                    | v1.x — rapidly evolving    |
| Zod v4              | https://zod.dev                                           | Confirm v4 tab is selected |
| Tailwind v4         | https://tailwindcss.com/docs                              | v4 branch                  |
| DaisyUI v5          | https://daisyui.com/docs/                                 | Confirm v5 (not legacy v4) |
| Alpine.js           | https://alpinejs.dev                                      | v3 docs                    |
| Iconify Tailwind4   | https://iconify.design/docs/usage/css/tailwind/tailwind4/ |                            |
| Lucide icon browser | https://icon-sets.iconify.design/lucide/                  | Look up exact icon names   |
| Hono v4             | https://hono.dev/docs/                                    |                            |
| Auth.js             | https://authjs.dev/reference/core                         | Hono uses @auth/core       |
| HTMX v2             | https://htmx.org/docs/                                    |                            |
| Drizzle ORM         | https://orm.drizzle.team/docs/overview                    |                            |
| Bun                 | https://bun.sh/docs                                       |                            |

**Web search capabilities** are available in this project. Prefer it over static `@Add Docs` for rapidly evolving packages like Mastra — it fetches live documentation mid-session.

---

## Feature-Scoping Checklist

Answer these five questions before starting any build session. Keep scope to one feature per session.

1. **What is the single user-facing action this feature enables?**
   (One sentence. If you need "and", split into two sessions.)

2. **Which Tier 2 docs are required?**
   (List the specific doc URLs to attach — no more than three.)

3. **Which existing source files should be attached as context?**
   (List specific files, not entire directories.)

4. **What is the Definition of Done?**
   (A specific HTTP response, UI state change, test assertion, or CLI output.)

5. **Is this scoped to one route group OR one agent capability — not both?**
   (If both are needed, split into two sessions: routes first, then agent integration.)

---

## Testing Strategy

- **Test runner:** `bun test` (Jest-compatible, built-in)
- **Balanced cadence:** during feature work, run targeted tests for the behavior you changed; before merge or milestone completion, run full `bun test`
- **Automation now:** use `bun test --watch` as the default local feedback loop while building
- **Automation later:** defer pre-commit/pre-push hooks and CI enforcement until a later stabilization milestone
- **DOM testing:** `happy-dom` — import in test setup for HTML fragment assertions
- **File convention:** `src/foo/bar.test.ts` colocated with `src/foo/bar.ts`

### Where Tests Live

- `src/index.tsx` -> `src/index.test.ts`
- `src/db/queries/notebooks.ts` -> `src/db/queries/notebooks.test.ts`
- `src/lib/foo.ts` -> `src/lib/foo.test.ts`
- For new domains, colocate tests with the source file they verify.

| Test Type      | Approach                                                                       |
| -------------- | ------------------------------------------------------------------------------ |
| Zod schemas    | `schema.parse()` and `schema.safeParse()` with valid + invalid fixtures        |
| DB queries     | Drizzle against in-memory libSQL (`:memory:` URL) with migrations run in setup |
| Hono routes    | `app.request()` helper — no network required                                   |
| Mastra tools   | Mock tool input/output contracts; test schema validation                       |
| HTMX responses | Parse HTML fragments with happy-dom; assert DOM structure                      |
| Responsive UI  | Verify key views at mobile + desktop breakpoints (`sm`, `md`, `lg`)            |

### TDD Guidance (Practical, Not Dogmatic)

- Prefer test-first (red -> green -> refactor) for route behavior, DB query contracts, and bug fixes.
- Allow short implementation-first spikes for UI exploration when behavior is still unknown.
- Before merge, convert spikes into assertions by adding or updating targeted tests for all changed behavior.
- Keep test scope tight: start with the failing behavior, then expand only if adjacent regressions are likely.

### Test Trigger Checklist

- Add a new `*.test.ts` file when you add a new module with meaningful behavior (route group, query helper, schema set, tool/workflow).
- Update an existing test file when you modify behavior in that module.
- Add a regression test for every bug fix.
- During implementation, run targeted tests for touched paths; before merge/milestone completion, run full `bun test`.
- If behavior changed and no assertion changed, that is usually a signal tests are missing.

---

## Hallucination Risk Summary

These packages have the highest risk of incorrect code generation. Always verify against docs before accepting suggestions:

1. **Mastra v1.x** — LLMs trained on pre-1.0 beta; all prior examples are likely invalid
2. **Zod v4** — Breaking changes from v3 in schema inference, `.default()`, error maps
3. **Tailwind CSS v4** — No JS config; CSS-first; `@plugin` directive is new
4. **DaisyUI v5** — Component class names changed; `@plugin` registration in CSS
5. **Iconify @iconify/tailwind4** — New CSS-only API; use a real two-class example like `iconify lucide--home` (canonical guidance lives in `.agents/skills/ui/SKILL.md`)
6. **Alpine.js v3** — Easy to confuse with Vue/Stimulus syntax; verify directives (`x-data`, `x-show`, `x-model`, `x-on`, `x-bind`) and magic properties before suggesting code
7. **HTMX v2** — Event syntax changed; some attributes removed
8. **Drizzle ORM 0.45** — Rapidly evolving; query builder API diffs from 0.3x

---
> Source: [PopPopRetired/PopPopLM](https://github.com/PopPopRetired/PopPopLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
