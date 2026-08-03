## wortschatz

> Operational notes for AI assistants working in this repo.

# CLAUDE.md

Operational notes for AI assistants working in this repo.

## Boundary: which surface gets the new code

**Read this before adding any route, endpoint, or server action.** Two HTTP
surfaces exist and the split is a hard rule, not a preference — full details
in [ARCHITECTURE.md](./ARCHITECTURE.md), deployment context in
[MONOREPO.md](./MONOREPO.md#api-boundary-rules).

- **Express (`apps/api`)** owns anything that **calls an LLM provider**, takes
  **> 5 seconds**, **processes binary data** (images/audio/PDF), or is a
  background job. All three AI endpoints live here: `/ai/review-text`,
  `/ai/evaluate-answer`, `/ai/generate-exercise`.
- **Next.js (`apps/web`)** owns auth/sessions, lightweight DB CRUD, read-only
  queries, session-bound server actions, and UI.
- **No file under `apps/web/src/` may import `@anthropic-ai/sdk` or `openai`.**
  CLI scripts under `apps/web/scripts/` may (offline-fallback generators
  only). A static test (`apps/web/src/__tests__/architecture.test.ts`) fails
  the build if this is violated.
- When Next.js needs an LLM result it calls Express via
  `src/lib/api-client.ts`. Schemas/logic both tiers need live in a shared
  package (`@wortschatz/exercises`), never in `apps/web/src/`.

## Stack

- **pnpm workspaces + Turborepo 2.x** monorepo
- **`apps/web`** — Next.js 15 (App Router, React 19 RC) + MUI v9 + emotion
- **`apps/api`** — Express 4 + helmet (ESM, runs via `tsx`)
- Shared packages: **`@wortschatz/{database,types,config}`**
- Tailwind CSS v4 — layout-utility classes only (see coexistence rule)
- next-intl for i18n (`en`, `pt`, `tr`, `uk`)
- Prisma + PostgreSQL, NextAuth.js v5
- Fonts: `next/font/google` → **Fraunces** (display) + **Inter** (body)
- vitest + jsdom + `@testing-library/react` for tests

All Next.js routes nest under `apps/web/src/app/[locale]/…`. The root
layout is minimal — locale HTML lives in
`apps/web/src/app/[locale]/layout.tsx`.

When this file refers to a path beginning with `src/…` (e.g.
`src/theme/`), read it as `apps/web/src/…` — the rules predate the
Sprint 03 monorepo split. See [MONOREPO.md](./MONOREPO.md) for the full
layout, deployment story, and current limitations.

## Monorepo conventions

- **Prefer the package barrel over the in-tree alias.** Use
  `import { prisma } from "@wortschatz/database"` (not `@/lib/db`),
  `import { MUENZEN_REWARDS, pickLocalized } from "@wortschatz/config"`
  (not `@/config/limits` or `@/lib/exercises/i18n`), and
  `import type { LocalizedText } from "@wortschatz/types"`. The old
  in-tree modules were deleted in Sprint 03; their re-imports won't
  resolve. Prisma enums are re-exported from `@wortschatz/database`,
  so never import directly from `@prisma/client`.
- **Scripts run from the repo root.** `pnpm dev`, `pnpm build`,
  `pnpm test`, `pnpm typecheck`, `pnpm db:generate`, `pnpm db:migrate`.
  Target a single workspace with
  `pnpm --filter @wortschatz/<name> run <script>`.
- **Shared values belong in `@wortschatz/config`** when both apps
  need them (Münzen rules, rate limits, locales, env schemas). Pure
  TS types belong in `@wortschatz/types`. Prisma schema, migrations,
  and the client singleton belong in `@wortschatz/database`. Don't
  duplicate.
- **One language per layer.** `apps/web` writes React + Next + MUI;
  `apps/api` writes Node + Express + zod. They share types but never
  cross-import (e.g. apps/api must never `import "@/something"`).

## Palette System + Material UI

All color, typography, radius, shadow, and shape tokens live in
`src/theme/`. Pages and components receive the theme via the
`<AppThemeProvider>` in `src/app/[locale]/layout.tsx`, which calls
`createAppTheme(mode)` from `@/theme`. **Never hardcode hex anywhere
outside `src/theme/`.**

- **Palettes.** `src/theme/palette.ts` defines both `lightPalette` and
  `darkPalette`. Mode selection is wired (see "Color modes" below).
- **Typography.** Headings use Fraunces via CSS var `--font-fraunces`,
  body uses Inter via `--font-inter`. MUI's `<Typography>` is the only
  way to set type — do not use Tailwind `font-*` classes.
- **Custom palette keys** (already augmented in `src/theme/augmentation.ts`):
  `tertiary`, `accentSoft`, `successSoft`, `dangerSoft`, `surfaceAlt`.
  Use them via `theme.palette.X` or MUI's `color="X"` where supported.
- **Hover behavior.** Buttons lift with `translateY(-1px)` + shadow on
  hover via the `MuiButton` style override in `src/theme/index.ts`. Never
  use `hover:opacity-90`.
- **Shape.** Inputs/buttons use the small radius from `shape`; cards use
  `RADIUS_CARD`; pills/chips use `borderRadius: 999` via MUI defaults.

### Color modes

- The user picks `'light' | 'dark'`. Default is `'light'` on first
  visit; persisted in `localStorage` under `wortschatz:color-mode`
  (exported as `COLOR_MODE_STORAGE_KEY` from
  `src/theme/ColorModeContext.tsx`). The pre-Sprint-04 `'system'`
  option was removed; old stored values map to `'light'` on read.
- Use `useColorMode()` from `@/hooks/useColorMode` to read or change
  the mode. It returns `{ mode, setMode, toggle }`. Don't access
  `localStorage` directly.
- **Hydration contract** (load-bearing — do not loosen):
  - The locale layout injects a blocking inline script in `<head>`
    that reads `localStorage` and writes
    `<html data-color-mode="…">` plus `style.color-scheme="…"` before
    React hydrates.
  - `<html>` carries `suppressHydrationWarning` for exactly those
    attributes (one-deep — body content still needs to match).
  - `AppThemeProvider` keeps its `useState` initializer pure (always
    `"light"` on both server and first client render) so emotion
    emits identical classNames; the user's actual stored choice is
    applied inside `useLayoutEffect`, which runs synchronously before
    the browser paints. Dark-mode users see one frame of dark, not a
    flash of light → dark.
- All new UI must work in both modes. Use `theme.palette.X` (or
  `sx={{ color: 'text.primary' }}` etc.) — never reference a specific
  palette mode in a component.
- The header `<ColorModeToggle />` is a 2-state sun ↔ moon icon with
  a stable localized `aria-label`. Rendered in both desktop and
  mobile menus.

For the full design system (palette tokens, typography scale,
component patterns, motion rules), see [DESIGN.md](./DESIGN.md).

## MUI ↔ Tailwind coexistence (mandatory rule)

**MUI owns** every component with state / variant / color / surface
semantics: Button, IconButton, Card/Paper, Chip, Avatar, TextField,
Dialog, Drawer, AppBar, Toolbar, Menu, Tooltip, Tabs, Typography. All
color, typography, radius, shadow, and shape values flow through the
theme.

**Tailwind owns** layout utilities only: `flex`, `grid`, `gap-*`,
`mx-auto`, `max-w-*`, `px-*`, `py-*`, `space-*`, `min-h-*`/`min-w-*`,
and responsive prefixes. Complex layouts can also use MUI `<Box sx={{}}>`
or `<Stack>` instead.

**Banned anywhere outside `src/theme/`**: hex colors, `bg-*`, `text-*`
(color variants — alignment utilities like `text-center`/`text-left` are
fine), `border-*`, `rounded-*`, `shadow-*`, `font-display`/`font-sans`/
`font-mono` Tailwind classes. Reach for the theme instead.

**Routing across the RSC boundary**: when a server page needs a
`<Button>` that navigates, use `<ButtonLink>` from
`src/components/ui/ButtonLink.tsx` — a client-side polymorphic shim that
wraps MUI's `Button` with the next-intl `Link`. Same for inline links
via `<InlineLink>`.

## Mobile-first responsiveness (mandatory)

- Works at 320 / 375 / 768 / 1024 / 1280+; no horizontal scroll.
- Touch targets ≥ 44×44px — enforced for `<IconButton>`, `<MenuItem>`,
  and `<ListItemButton>` via the theme defaults; size-medium `<Button>`
  is 44px too.
- Inputs always render at 16px (set in `MuiOutlinedInput` / `MuiInputBase`
  to defeat iOS Safari focus-zoom).
- Tailwind responsive prefixes only — no hardcoded widths.
- Wrap routed content in `mx-auto max-w-… px-4 sm:px-6`.
- Default 1 col on mobile; add columns at `sm:`/`md:`/`lg:`.
- Tables: `overflow-x-auto` **and** stacked card fallback.

## Shared UI primitives

Live in `src/components/ui/`: `MuenzenBadge`, `StreakFlame`, `LevelChip`,
`ExerciseTypeIcon`, `Card`, `ButtonLink`, `InlineLink`. Extend, don't
re-implement.

## Project layout

```
apps/
  web/                              Next.js 15 app
    src/
      app/[locale]/                 Routed pages, locale-prefixed
      components/layout/            Header, Footer, LocaleSwitcher, MobileMenu
      components/ui/                Shared design-system primitives
      components/exercises/renderers/   10 exercise type renderers
      components/dashboard/         Dashboard chart components (Recharts + SVG)
      theme/                        Palette: palette, typography, shape,
                                    shadows, augmentation, Provider
      test/                         vitest setup + renderWithTheme helper
      i18n/                         next-intl config + typed nav helpers
      lib/                          ai (thin api-client wrapper), muenzen,
                                    exercises/, review/, comments/, profile/
      lib/dashboard/                Pure aggregations + parallel Prisma queries
    scripts/                        Admin one-shots (generate-exercises, etc.)
    messages/*.json                 UI translations (incl. `renderers` block)
  api/                              Express 4 (ESM, tsx)
    src/
      index.ts                      App boot + middleware chain
      env.ts                        zod-validated process.env
      middleware/                   sharedSecretAuth, errorHandler, logger
      services/                     claude, cache, rateLimit, stubs
      routes/                       /health, /ai

packages/
  database/                         @wortschatz/database
    prisma/{schema.prisma, migrations/, seed.ts}
    src/index.ts                    PrismaClient singleton + re-exports
  types/                            @wortschatz/types (wire formats)
    src/{exercise,ai,user,muenzen,locale}.ts
  config/                           @wortschatz/config (constants, env, utils)
    src/{constants,env,validators,utils}.ts
```

## Exercise system rules

- **`ExerciseType` enum** is English in the DB. Display names come from
  `messages/*.json` (`exerciseTypes`, `exerciseTypeDescriptions`).
- **i18n content.** `Exercise.instructions` and `Exercise.explanation` are
  JSON `{ en, pt, tr, uk }` — read with `pickLocalized(value, locale)`.
  German `content`/`solution` stays in German regardless of locale.
- **Münzen are awarded only on the FIRST passing attempt** (`score ≥ 60`)
  per user+exercise. `submitExerciseAttempt` always records the attempt.
  As of Sprint 02 Task 4, the perfect-score bonus is written as a
  separate `PERFECT_SCORE_BONUS` transaction (not folded into
  `EXERCISE_COMPLETE`), and admins can adjust user balances via the
  `ADMIN_ADJUSTMENT` reason from `src/app/[locale]/admin/`.
- **No hardcoded English/German UI strings in renderers.** Labels live in
  the `renderers` block in `messages/*.json`. (German is allowed only for
  the _content being learned_.)

### Exercise intros

- Each exercise type has a static intro at
  `src/content/exercise-intros/<type>.ts`, aggregated into
  `EXERCISE_INTROS: Record<ExerciseType, ExerciseIntro>`. Each entry has
  `whatItAsks`, `howToInteract`, `example.prompt`, and
  `example.solvedExplanation`, all `LocalizedText` (en/pt/tr/uk). Edit
  those files to update copy — no AI calls.
- "Don't show again" is per-user-per-type, stored in the `UserPreference`
  table. Use `setSkipIntro` / `getSkipIntro` from
  `@/lib/preferences/actions` — never mutate the table directly.
- The intro is shown inline on the type runner (`/exercises/<TYPE>`) the
  first time; revisit later via the `?` keyboard shortcut (Shift+/,
  suppressed while focus is on `input`/`textarea`/`contentEditable`) or
  the help-icon button in the single-exercise header.
- `UserPreference` is the first row in a deliberately-narrow preference
  table. Its `key` column currently reuses `ExerciseType`; when more
  preference kinds arrive, promote `key` to a dedicated `Pref` enum
  rather than overloading `ExerciseType` further.

### Münzen transactions

- Every Münzen balance change writes a `MuenzenTransaction` row. Always
  use `credit` / `debit` / `adminAdjust` from `@/lib/muenzen` — never
  mutate `user.muenzen` directly.
- Reason enum: `EXERCISE_COMPLETE`, `PERFECT_SCORE_BONUS`, `DAILY_STREAK`,
  `SPENT_AI_REVIEW`, `ADMIN_ADJUSTMENT`, `BONUS` (legacy, retained).
- History UI lives at `/profile/historico` (same path across all four
  locales).

### Exercise generation (admin UI + CLI)

The `pnpm gen:claude` / `gen:gpt` CLIs and the admin generator at
`/admin/generate` share one engine: `runGeneration` in
`apps/web/scripts/shared/run.ts`. The web API imports it directly via the
`@scripts/*` path alias (in `tsconfig.json` + `vitest.config.ts`) — **no CLI
spawn**. Full details in [scripts/README.md](./apps/web/scripts/README.md).

- **Prompt-builder seam (locked vs editable).** Each per-type prompt file
  (`scripts/<provider>/prompts/<type>.ts`) exports `promptParts: PromptParts`
  with four pieces. `system` + `instructions` are the editable "voice" an
  admin may override; `jsonShape` + `rules` are **locked** — the runner always
  injects the canonical versions so a custom prompt can never break Zod
  validation. `shared/prompt-builder.ts` composes them, reproducing the legacy
  monolithic prompt byte-for-byte (pinned by `scripts/shared/__tests__/prompt-parity.test.ts`;
  regenerate the baseline only after an intentional prompt edit).
- **Session model.** Every non-dry run writes exactly one `GenerationSession`
  (Decision 8): `source` (`UI`/`CLI`), provider, model, type/level/topic,
  requested count, optional `savedPromptId`, `customSystem`/`customInstructions`
  flags, and the outcome (`successCount`/`failureCount`/`failures`/`durationMs`).
  Exercises link back via `Exercise.generationSessionId` (`onDelete: SetNull`).
  CLI runs author the session as the seed admin; UI runs as the acting admin.
  Sessions are an audit trail — there is no delete. Create/finalize via
  `shared/session.ts`; never write the table ad-hoc.
- **`SavedPrompt`** rows are per-admin prompt templates (private; Decision 3).
  `useCount` is bumped with an atomic Prisma `increment` each time a session
  uses one. Deleting a template keeps its sessions (`SetNull`).
- **Admin surface.** API routes under `apps/web/src/app/api/admin/`
  (generate-exercises, saved-prompts CRUD, preview-prompt, generation-sessions)
  are **ADMIN-only** (TEACHER excluded) and rate-limited through the existing
  `AiRateLimit` `GENERATE_EXERCISE` window. UI pages: `/admin/generate`,
  `/admin/prompts`, `/admin/generate/history[/:id]`. v1 exposes Claude only.

### Base prompts (DB-backed prompt curation)

Per-`(ExerciseType, CefrLevel)` generation prompts live in the **database**,
editable in production by ADMIN/TEACHER — the hardcoded per-type files are the
fallback. Full guide in [docs/prompts.md](./docs/prompts.md).

- **DB-first resolution.** At generation time `getActiveBasePromptVoice(type,
  level)` (in `@wortschatz/database`) returns the one `ACTIVE`
  `BasePromptVersion`, or `null` → the hardcoded file is used unchanged
  (Decision 5; the parity test stays green because the files are never edited).
  The pure `applyPromptVoice` (in `@wortschatz/exercises`) layers the stored
  `system`+`userInstructions` (with `{level}`/`{topic}` interpolation) onto the
  file's **locked** `jsonShape`/`rules`. Resolution happens on the Express
  generate service (live path) and the web CLI-offline generator. GPT always
  uses its file (curation is Claude-only in v1).
- **Versioning (append-only).** Editing creates a new `DRAFT`; **publish** flips
  it to `ACTIVE` and demotes the prior `ACTIVE` to `INACTIVE` in one
  transaction; **revert** reactivates an `INACTIVE` row. Versions are never
  deleted. `versionNumber` is monotonic **per `basePromptId`**, not global.
  `Exercise.basePromptVersionId` (`SetNull`) traces each exercise to its prompt.
- **Roles.** TEACHER can list/draft/test/publish; **revert is ADMIN-only**.
  `jsonShape`/`rules` are code-locked for everyone (no admin-only editable
  field) — so there's nothing to strip for TEACHER on draft create. Use
  `requireAdminOrTeacher()` for the curation routes (under
  `apps/web/src/app/api/admin/base-prompts/`), `requireAdmin()` for revert.
- **test-generate** spends real tokens (counts against `GENERATE_EXERCISE`,
  writes `AiUsage` with `source="test-generate"`) but inserts nothing and writes
  no `GenerationSession`. The 40 seed rows + v1 content live in
  `packages/database/prisma/seed-data/base-prompts.ts`; the seed is idempotent.

## Testing

`vitest` is the runner; tests live under `__tests__/` directories beside
the source they cover. From the repo root run `pnpm test` (turbo runs
every workspace's vitest), `pnpm --filter @wortschatz/web run test`
(just the web suite), or `pnpm --filter @wortschatz/web run test:watch`
for watch mode. Theme + UI primitives are covered as of Sprint 02
Task 1. Use the `renderWithTheme` helper from `apps/web/src/test/` to
mount components inside the real theme. Color-mode tests live in
`apps/web/src/hooks/__tests__/useColorMode.test.tsx`,
`apps/web/src/theme/__tests__/Provider.test.tsx`, and
`apps/web/src/components/layout/__tests__/ColorModeToggle.test.tsx`.

`apps/api` has no test suite yet — the `test` script uses
`--passWithNoTests` so `pnpm test` doesn't fail. A follow-up sprint
should port the cache / rate-limit / claude unit tests from the
pre-migration `apps/web/src/lib/__tests__/` over.

## Dashboard charts

- The dashboard pulls all chart data through
  `fetchDashboardChartData(userId, now)` in `src/lib/dashboard/queries.ts`.
  New charts must funnel through the same parallel batch — don't add
  ad-hoc Prisma calls on the dashboard page.
- Pure aggregations live in `src/lib/dashboard/aggregations.ts`. Tests
  pin every branch. If you add a new chart, add a pure helper alongside.
- Charts that use Recharts must be split into a base file (`*Chart.tsx`)
  and a `.client.tsx` `next/dynamic({ ssr: false })` wrapper. The server
  page imports the `.client.tsx` only.
- Colors come from `theme.palette` exclusively. No hex anywhere in
  `src/components/dashboard/` or `src/lib/dashboard/`.

## User model

The `User` row carries profile + preferences alongside the auth fields.
As of Sprint 02 Task 6 it includes `bio` (string, max 280),
`nativeLanguage` (string, ISO code), `learningLevel` (`CefrLevel?`),
`dailyGoal` (`Int`, default 5), and `avatarUrl` (string, optional).
The dashboard's daily-goal ring reads `user.dailyGoal ?? DAILY_GOAL_DEFAULT`
and the review page reads `user.learningLevel ?? 'B1'` — no more
hardcoded CEFR levels.

## Avatar uploads

- **Endpoint.** `POST /api/profile/avatar` (auth required,
  `multipart/form-data`, field name `file`). `DELETE` on the same path
  removes the current avatar.
- **Storage.** Local filesystem at `public/uploads/avatars/`. Fine for
  dev and simple self-hosted; **must be swapped for Vercel Blob or S3
  before deploying on an ephemeral filesystem** (Vercel/Netlify). The
  swap site is marked with a `TODO` in the route handler.
- **Pipeline.** All processing lives in `src/lib/profile/avatar.ts`:
  size check (2 MB cap) → mime allowlist (jpeg/png/webp/gif) → spoof
  defense via `sharp().metadata()` re-decode → rotate → resize 512×512
  cover → WebP encode (quality 88) → write with random 6-byte hex
  filename suffix. Output is always 512×512 WebP regardless of input.
- **Path traversal hardening.** The route handler rejects any filename
  segment containing `/` or `..` after the reviewer pass.

## Comments + likes

- User-generated content lives in `ExerciseComment` rows. Always render
  via React text children — **never** `dangerouslySetInnerHTML`. Use
  `whiteSpace: 'pre-wrap'` (or the equivalent `sx`) to preserve newlines.
- Soft delete via `deletedAt`. `serializeComment` in
  `src/lib/comments/serialize.ts` masks both content and author on
  deleted rows; `loadComments` / `loadComment` in `queries.ts` also
  filter `deletedAt: null` so deleted rows never escape the boundary.
- Moderation knobs live in `@wortschatz/config`:
  `COMMENT_WORD_BLOCKLIST` (currently empty — new bad words go here),
  `COMMENT_MAX_LENGTH` (500), `COMMENT_RATE_LIMIT` (5 per 60 s), and
  the `findBlockedWord` helper. The matcher normalizes whitespace and
  lowercase before comparing.
- Rate limit: 5 comments per 60 s per user, counted directly from the
  `createdAt` column on `ExerciseComment` (no separate counter table).
- Admins can soft-delete other users' comments but **cannot edit them** —
  `PATCH /api/comments/[id]` is author-only (403 for admins editing
  someone else's row). `DELETE /api/comments/[id]` allows author OR
  admin.
- Like toggle is race-safe via `prisma.$transaction` + a P2002-catch on
  `(commentId, userId)`; the route returns `{ liked, likeCount }`.

## Multi-agent workflow (Sprint 02+)

Each task flows `@coder` → `@reviewer` → `@tester` → `@docs`. Coder
writes production code; reviewer audits (no tests, no docs); tester
writes tests (no production code); docs updates `CLAUDE.md`,
`SPRINT_02.md`, `ROADMAP.md`, etc. One semantic commit per task.

## AI integration

As of Sprint 03 the Claude pipeline is split:

- **`apps/api/src/services/claude.ts`** owns `reviewText` and
  `evaluateAnswer` (the user-facing endpoints). It's the only place
  inside apps/api that imports the Anthropic SDK.
- **`apps/web/src/lib/ai.ts`** still owns `generateExercise` — the
  admin script `apps/web/scripts/generate-exercises.ts` calls it
  directly via the in-process Anthropic SDK because the per-type Zod
  schemas haven't been extracted into a shared package yet.
- **`apps/web/src/lib/api-client.ts`** is the web's only path to
  `reviewText`/`evaluateAnswer`. `src/lib/ai.ts` re-exports the
  client's functions under the original names so callers in
  `review/actions.ts`, `exercises/actions.ts`, etc. keep their
  existing import paths.

Boundary auth (web → api): every call ships `X-Internal-Secret`
(constant-time compared against `INTERNAL_API_SECRET`) plus
`X-User-Id` (resolved from the NextAuth session on the web server
before the request leaves the process). Anonymous/admin calls omit
the user header; the api then skips per-user rate limits. See
[MONOREPO.md](./MONOREPO.md#web--api-boundary) for the rationale.

Constants, TTLs, rate limits, pricing:

- All live in `@wortschatz/config` — both apps import them from there.
- **Stubs.** When the api has no `ANTHROPIC_API_KEY` the
  review/evaluate routes return deterministic stub bodies and write
  nothing to the DB. When the web has no key the in-process
  `generateExercise` does the same; the stub for the exercise renderer
  lives in `apps/web/src/lib/ai-stubs.ts`.
- **Caching.** SHA-256 over `endpoint:model:canonicalPrompt`. TTLs
  live in `AI_CACHE_TTL_MS` (`REVIEW_TEXT: 0` = never cached;
  `EVALUATE_ANSWER: 1h`; `GENERATE_EXERCISE: 30 days`). `set` swallows
  errors and a TTL of 0 skips the write entirely; expired rows are
  best-effort deleted on read.
- **Rate limits.** Per-user, per-endpoint, rolling 24h window via
  `prisma.$transaction`. Limits in `AI_RATE_LIMITS` (`REVIEW_TEXT:
  20`, `EVALUATE_ANSWER: 200`, `GENERATE_EXERCISE: 50` per day).
  Cache hits do not count.
- **Errors.** Catch `AiRateLimitedError` at the call site and surface
  a localized message (review returns `{ ok: false, error:
  'rate_limited' }`; evaluate falls back to the local grade and
  still records the attempt). The api translates 429 → that same
  error class so the catch site doesn't care whether the limit fired
  in-process or remotely.
- **Cost.** `estimateCostMicrocents(model, inputTokens, outputTokens)`
  in `@wortschatz/config` returns integer microcents (1 cent = 100 µ¢;
  USD × 100 000). Persist on `AiUsage.costMicrocents` to avoid float
  drift.
- **Observability (Helicone).** Every Anthropic/OpenAI client supports
  optional routing through the [Helicone](https://helicone.ai) proxy for
  prompt/response/cost logging. It's gated on the optional
  `HELICONE_API_KEY` env var: when set, clients construct with the
  `baseURL` + headers from `heliconeAnthropicOverrides` /
  `heliconeOpenAIOverrides` (and per-request `heliconeRequestHeaders`) in
  `@wortschatz/config`; when unset those helpers return `{}` and behavior
  is identical to before. New LLM client instantiations should spread the
  matching helper rather than inline the proxy config. Helicone is purely
  additive — `AiUsage` stays the canonical internal log. Details in
  [scripts/README.md](./apps/web/scripts/README.md#observability-with-helicone).

## What NOT to change without a request

- Prisma schema, API routes, grading, Münzen amounts
- AI surface (`apps/web/src/lib/ai.ts` re-exports +
  `apps/api/src/services/claude.ts`) — keep the public function
  signatures stable; if you change prompts, follow the cache
  invalidation note in `SPRINT_02.md` (the cache key includes the
  prompt body, so old rows for the old prompt remain until their TTL
  expires).
- `@wortschatz/config` constant values (Münzen rules, rate limits,
  TTLs, pricing) — both apps depend on them being equal; bump in one
  place, not via local override.
- `runGeneration`'s public shape (`scripts/shared/run.ts`) and the
  per-type `promptParts` JSON-shape/rules — the CLI and the admin UI both
  depend on them; the prompt-parity test guards against silent drift. The
  `GenerationSession` table is the long-term foundation for curation: don't
  denormalize it or skip writing it for performance without flagging it.
- The hardcoded per-type prompt files are the **fallback of record** for
  DB-backed curation — don't edit their text to "fix a prompt" (edit the DB
  version via `/admin/prompts/base`), and never remove the file fallback or the
  locked `jsonShape`/`rules` seam. `BasePromptVersion` is append-only: don't add
  an update/delete path or fold `jsonShape`/`rules` into the DB. See
  [docs/prompts.md](./docs/prompts.md).

---
> Source: [JonathanRomano/wortschatz](https://github.com/JonathanRomano/wortschatz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
