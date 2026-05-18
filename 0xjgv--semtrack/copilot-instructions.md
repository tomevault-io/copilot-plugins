## semtrack

> Security vulnerability tracker for Semgrep scan results — Next.js 16, Bun, PostgreSQL + Drizzle ORM, Mastra AI agent, Tailwind v4 + shadcn/ui, Biome.

# CLAUDE.md

Security vulnerability tracker for Semgrep scan results — Next.js 16, Bun, PostgreSQL + Drizzle ORM, Mastra AI agent, Tailwind v4 + shadcn/ui, Biome.

## Project map

- `app/` — Next.js App Router pages and API routes (`app/api/`)
- `lib/db/` — Drizzle ORM schema (`schema.ts`) and data access layer
- `lib/mastra/` — Mastra AI agent, tools, and semantic memory
- `lib/ai/` — AI provider config (OpenAI-compatible, configurable via env)
- `lib/hooks/` — SWR data fetching and mutation hooks
- `lib/` — Domain logic: types, fingerprinting, severity normalization, ingest pipeline
- `components/ui/` — shadcn/ui primitives; `components/` — custom components
- `tests/` — Bun test files
- `fixtures/` — Semgrep scan JSON fixtures (excluded from tsc and Biome)

<important if="you need to run commands to build, test, lint, or check code">

| Command | What it does |
|---|---|
| `bun run check` | Fix, format, typecheck, test (run after edits) |
| `bun run pre-commit` | Staged files only (auto via git hook) |
| `bun run ci` | Read-only lint, typecheck, dep audit, tests with coverage |
| `bun run db:push` | Apply schema changes to the database (Drizzle push, no migrations) |
| `bun harness.ts setup` | Create .env.local, install deps, start Postgres, push schema |
| `bun harness.ts help` | Show all available commands |

</important>

<important if="you are adding or modifying imports">
- Use `@/` path alias for all imports (maps to project root)
</important>

<important if="you are adding UI components or icons">
- Add shadcn components: `npx shadcn@latest add <name>`
- Icons: `@phosphor-icons/react` (not Lucide)
</important>

<important if="you are writing or modifying data access functions in lib/db/">
- All mutations must write an `auditEvents` record within the same transaction
- `lib/domain-types.ts` is the single source of truth for domain types, statuses, transitions, and constants
- AI tools in `lib/mastra/tools.ts` call the same `lib/db/` functions — one code path for all mutations
- Schema is push-based (no migration files) — edit `lib/db/schema.ts` then run `bun run db:push`
- All timestamps are Unix milliseconds stored as `bigint` (`mode: "number"`) — not `Date` objects
- Bulk operations return `{ updated, skipped }` — never throw for partial failures
</important>

<important if="you are adding or modifying API routes in app/api/">
- `params` is a `Promise` and must be awaited (Next.js 15+ dynamic API)
- Routes are thin — delegate to `lib/db/` functions, no business logic in handlers
- `repoSlug` is the public identifier for repos (not `repoId`) — resolve via `getRepoBySlug()`
- Mutation routes call `getCurrentUser()` and pass `actor`/`actorType` to db functions
- No auth middleware — identity is a `semtrack_user_id` cookie via `lib/api/current-user.ts`
</important>

<important if="you are modifying client-side data fetching or cache invalidation">
- SWR handles all client-side caching — no `revalidatePath`/`revalidateTag` in this codebase
- After mutations, call `revalidateAppData(mutate)` from `lib/swr-helpers.ts` to invalidate SWR cache
- Read hooks return named aliases (`findings`, `repos`) defaulted to `[]` — never raw `data`/`undefined`
</important>

<important if="you are writing tests or working with fixture data">
- `fixtures/` is inert test data — never import from application code
- Tests run via `bun test`; files live in `tests/`
- For DB-dependent tests, `mock.module()` must be called before importing the module under test
</important>

<important if="you are working on the AI agent or chat feature">
- Agent definition: `lib/mastra/agent.ts`; tools wrap shared `lib/db/` functions
- AI model configurable via `AI_API_KEY`, `AI_BASE_URL`, `AI_MODEL` env vars
- Mastra packages are `serverExternalPackages` in `next.config.mjs`
- Mastra tables live in the `mastra` PostgreSQL schema (not `public`) — Drizzle manages only `public` via `schemaFilter`
- AI tools return errors as `{ error: string }` objects — they never throw
</important>

<important if="you are working on scan ingest or upload">
- Two-layer pipeline: `lib/ingest.ts` (pure transform) → `lib/db/scans.ts#ingestScan` (transactional upsert)
- Advisory lock on repo ID prevents race conditions on concurrent uploads
- Scan dedup via `contentHash` — duplicate uploads return HTTP 409
</important>

---
> Source: [0xjgv/semtrack](https://github.com/0xjgv/semtrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
