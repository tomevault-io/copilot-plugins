## jant

> Jant (short for Jantelagen) is a personal microblogging system — self-hosted, single-author, and stripped of all social mechanics. No followers, no likes, no algorithmic feed. It combines Tumblr-style multi-format posts (notes, links, quotes), Threads-style threading for connected thoughts, and curated Collections to organize content by topic. Just a clean space for one person to think out loud.

# Jant - Development Guide

## What is Jant

Jant (short for Jantelagen) is a personal microblogging system — self-hosted, single-author, and stripped of all social mechanics. No followers, no likes, no algorithmic feed. It combines Tumblr-style multi-format posts (notes, links, quotes), Threads-style threading for connected thoughts, and curated Collections to organize content by topic. Just a clean space for one person to think out loud.

It runs on Cloudflare Workers with minimal infrastructure. The UI follows an "Organic Minimalism" aesthetic: generous whitespace, single-column layout, smooth animations, mobile-first. Content comes in three formats — Note (), Link (shared reference), Quote (cited text) — organized through Threads and Collections.

The project is in **pre-1.0 development**. Breaking changes are expected and welcome when they improve the design. Always follow best practices over minimal-change conservatism. Update all references in the same change and document what changed in the commit message.

## Workflow Orchestration

### 1. Plan Mode Default

– Enter plan mode for ANY non-trivial task (3+ steps)
– If something goes sideways, STOP and re-plan immediately
– Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy

– Use subagents liberally to keep main context window clean
– One task per subagent for focused execution
– For complex problems, throw more compute at it

### 3. Self-Improvement Loop

– After ANY correction: update tasks/lessons.md
– Write rules that prevent the same mistake
– Ruthlessly iterate until mistake rate drops

### 4. Verification Before Done

– Never mark a task complete without proving it works
– Ask yourself: “Would a staff engineer approve this?”
– Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)

– Pause and ask “is there a more elegant way?”
– Skip this for simple fixes — don’t over-engineer

### 6. Autonomous Bug Fixing

– When given a bug report: just fix it
– Zero context switching required from the user

## Task Management

1. Plan First: Write plan to tasks/todo.md
2. Verify Plan: Check in before starting
3. Track Progress: Mark items complete as you go
4. Explain Changes: High-level summary at each step
5. Document Results: Add review section to todo.md
6. Capture Lessons: Update lessons.md after corrections

## Development Philosophy

These principles explain _why_ the codebase is structured the way it is. When you encounter a situation not covered by a specific rule, use these to guide your judgment.

- **Challenge before complying**: when the user proposes an approach that conflicts with best practices or this document, push back with a clear explanation of the trade-offs and ask for confirmation before proceeding. Silently following a suboptimal instruction is worse than a brief discussion.

- **Separation of concerns**: routes handle HTTP, services own business logic and all DB access, UI renders data. Each layer should be replaceable without affecting the others. Module dependency direction: `routes → services → db`, `routes → viewmodels → ui`. Detailed rules in `docs/internal/coding-standards.md`.

- **Routes are thin adapters**: a route handler should only parse/validate the request, call one or more service methods, and format the response. Multi-service orchestration (e.g. "delete a post and clean up its media files") belongs in the service layer, not in routes. **Litmus test**: if two routes need the same sequence of service calls, that sequence must be extracted into a service method. Cross-cutting concerns like storage file cleanup are passed to services via optional dependency parameters (e.g. `storage?: StorageDriver | null`) rather than being handled in routes.

- **Type safety as communication**: TypeScript strict mode with no `any` and fully typed exports prevents silent contract drift between layers. When a service return type changes, the compiler should catch every consumer.
- **Normalize interpolated copy inputs before rendering**: when user-facing copy interpolates labels, hosts, dates, counts, or settings, normalize the value first. Empty or whitespace-only strings must fall back explicitly before they reach the UI. Do not rely on truthiness fallbacks for numeric or boolean values because `0` and `false` are often valid data. In translated copy, never bake runtime values into the `message` string itself; use placeholders plus `values` so extraction, translation, and fallback behavior stay correct.
- **Hosted integrations stay neutral in core**: when `jant-core` integrates with a hosted control plane, keep the runtime and copy provider-neutral. Brand names belong in the control plane, not in core. Core may show a configured provider label, and when no label is configured it should fall back to the provider host/domain instead of hardcoding product branding.
- **Hosted control-plane metadata is a projection, not a second truth source**: hosted site display metadata belongs to `jant-core`. Any control-plane copy (for example cached site name or primary host shown in a dashboard) must be treated as a denormalized projection synced from core, never as an independently editable second source of truth.
- **Separate public URLs from internal URLs**: hosted control-plane redirects used by browsers should go through the public control-plane URL. Server-to-server calls should use explicit internal URLs and tokens when available instead of reusing public entrypoints by name.
- **Hosted site lifecycle belongs to the control plane**: billing state, cancel/delete/restore policy, and retained-window rules belong in `jant-cloud`. `jant-core` may link to those flows and expose internal execution APIs, but it must not invent separate hosted billing or deletion semantics of its own.

- **Tokens and components over raw values**: CSS tokens (`styles/tokens.css`) and BaseCoat semantic classes (`.alert`, `.btn`, `.badge`, `.card`, `.input`, `.field`) encode design decisions in one place. Hardcoding a color or spacing value means it can't evolve with the theme. See `docs/internal/theming.md` and `references/basecoat/`.

- **Cohesion over small files**: organize code by responsibility and keep related logic together. A well-structured 400-line file is better than four fragmented 100-line files that constantly import each other.

- **Strict boundaries, free internals**: validate and convert at boundaries (HTTP entry, DB queries). Once data is inside a layer, trust the types.

- **Data flows down**: DB → Service → ViewModel → Component. Never in the other direction.

- **Fail fast**: missing required config should crash at startup with a clear error, not silently degrade at runtime.
- **External links should be intentional**: links that open in a new tab must include `rel="noopener noreferrer"`. Do not add `nofollow` to normal editorial links by default; reserve it for ads, sponsored placements, or future user-generated content.

- **Keyboard-first interactions**: every dialog, panel, and overlay must support standard keyboard shortcuts — `Escape` to close/cancel, `Enter` to confirm the primary action, `Tab` for focus navigation. Never rely solely on mouse/touch. Note that `<dialog>` native cancel events may not fire when inner elements (e.g. TipTap/ProseMirror) intercept `Escape` at the keydown level; always handle keyboard events directly on the component in addition to native dialog events.
- **Dismiss transient UI predictably**: menus, popovers, dropdowns, and other temporary overlays must close on outside click/tap and `Escape`, and opening one should close peers of the same kind. Do not rely on browser defaults like `<details>` dismissal behavior; wire dismissal explicitly and cover it with a test.
- **Preserve interaction affordances**: custom-styled interactive controls must still feel interactive. Ensure clickable elements expose expected affordances such as `cursor: pointer`, visible hover/focus states, sensible disabled states, and correct initial focus when opening dialogs or overlays.

### Hard Constraints

Non-negotiable regardless of context:

- **No DB in routes**: routes must never contain direct DB calls, raw SQL, or import DB drivers. All data access goes through `src/services/`.
- **No business logic in routes**: routes must not orchestrate multi-service operations, coordinate side effects, or duplicate logic that belongs in a service. If two routes would need the same logic, it must live in a service method.
- **Migrations are append-only and schema-first**: never edit or replace an existing migration file in `src/db/migrations/`. Applied migrations are tracked by filename — changing their content causes drift between the migration history and the actual database state, breaking local and production environments. Always create a new migration file for schema changes, even if the previous migration was recently added. For example, to undo migration `0016`, create `0017` with the reverse DDL rather than rewriting `0016`. For normal schema changes, update `src/db/schema.ts` first, then run `drizzle-kit generate` (via `mise run db-schema-generate`) to produce the migration file. Rare manual schema migrations are allowed only when Drizzle cannot express the schema object, such as FTS virtual tables or triggers. Keep those exceptions in `src/db/migrations/`, use the same `0000_name.sql` numbering, and update the journal/snapshot metadata in the same change. Historical data compatibility fixes are a separate track: put them in `src/db/backfills/` as append-only numbered SQL files, make them idempotent, run them via `jant migrate`, and never insert ad-hoc files like `0004z_*` into `src/db/migrations/`.
- **Dual-dialect schema: SQLite AND Postgres must stay in sync**: Jant runs on both SQLite/D1 (`src/db/schema.ts`) and Postgres (`src/db/pg/schema.ts`). When adding or modifying columns, **always update both schema files**. Migrations also exist in two directories: `src/db/migrations/` (SQLite/D1) and `src/db/migrations/pg/` (Postgres). Generate SQLite migrations with `mise run db-schema-generate` and Postgres migrations with `mise run db-schema-generate-pg`. If auto-generation produces "no changes", write the Postgres migration manually and update its `meta/_journal.json`. Forgetting to update the Postgres schema will cause silent data loss — Drizzle will ignore columns it doesn't know about.
- **Seed/import SQL must declare columns and validate against current schema**: committed SQL snapshots and export scripts must use `INSERT INTO table (col, ...) VALUES (...)`, never bare `INSERT INTO table VALUES (...)`. After schema changes, validate seed/import SQL against a fresh local D1 with the current migrations before treating it as safe.
- **Relative imports only**: no `@/` path aliases anywhere in the codebase.
- **Data attributes with care**: `data-page`, `data-post`, `data-format`, etc. are consumed by themes and external scripts. Design them thoughtfully and update all references when changing.
- **No raw strings in `dangerouslySetInnerHTML`**: every string passed to `dangerouslySetInnerHTML` must be either (a) HTML produced by a trusted renderer (TipTap, `getHtmlExcerpt`), or (b) plain text that has been passed through `escapeHtml()` first. Never concatenate plain-text fields into HTML strings without escaping. When injecting highlight markers into escaped text, use `escapeHtml()` first, then replace control-character sentinels with `<mark>` tags — never inject `<mark>` directly into unescaped text.

## Working with the Codebase

### Tooling

- **Use mise tasks** for all commands (`mise tasks` to list). Never run `wrangler dev`; use `mise run dev` / `mise run build`.
- **Deployment shorthand**: when the user says "部署", interpret it as "commit the current work and push the current branch". Site rollout happens automatically after push unless the user says otherwise.
- **Debug**: `mise run dev-debug` prepares local auth helpers automatically and uses the first free debug port starting at `19020`. For browser testing, use the printed `http://localhost:19xxx/__dev/login?token=...&redirect=/settings` URL with `DEV_API_TOKEN` from `packages/core/.dev.vars`, then continue on `http://localhost:19xxx/settings`. `jant.localtest.me` is still accepted locally, but some browsers upgrade it to HTTPS and break local HTTP dev ports. HTTP agents can call the same local login URL directly and reuse the returned `Set-Cookie`. Stop background processes when done.
- **Verify before changing**: never assume CLI flags; confirm with `--help` or docs.
- **Latest packages**: use `@latest` when installing.
- **Generated template is read-only**: never edit `packages/create-jant/template/`.
- **GitHub Actions**: always add `workflow_dispatch:`.
- **Verify proportionally**: choose verification based on the risk and surface area of the change instead of mechanically running the full suite every time.
  - Run `mise run check-tests` and `mise run check-lint` for behavior changes: routes, services, DB/schema/migrations, validation, auth, build tooling, shared infrastructure, interactive client logic, or anything with meaningful regression risk.
  - For isolated visual or content-only changes, such as CSS-only tweaks, spacing, color, typography, copy, or docs, use judgment. A focused sanity check is usually enough if no logic, markup structure, or event handling changed.
  - If a change sits near the boundary, prefer the narrower relevant verification first, then escalate to full `check-tests`/`check-lint` if the impact is broader than expected.
  - Always state what you verified, and explicitly note when you skipped automated checks.

### Conventions

- `packages/core`: library + dev environment (Vite HMR). `sites/demo`: demo site + user template source (via `@create-jant` annotations).
- **Types**: public exports in `src/types.ts`; definitions in `src/types/`.
- **Schemas**: shared domain schemas in `src/lib/schemas.ts`; route-specific schemas colocated with routes.
- **Routes**: `xxxRoutes` suffix (`postsRoutes`, `settingsRoutes`).
- **DB table names**: always singular or domain-specific (`post`, `collection`, `nav_item`, `api_token`, `path_registry`), never plural.
- **Time**: Unix timestamps (seconds) via `lib/time.ts`.
- **IDs**: TypeID (text) everywhere — DB, API, auth tables, jobs, storage-backed entities. Store canonical TypeID strings directly; never convert them back to UUIDs in application code. Use the shared prefixes from `src/lib/ids.ts` (`pst`, `med`, `col`, `pth`, `cdi`, `nav`, `api`, `usr`, `ses`, `acc`, `vrf`). **Post URLs**: slug-based (`/{slug}`). Slugs are auto-generated from title (via `lib/slug.ts`) or as random alphanumeric IDs (via `lib/nanoid.ts`). Path overrides are managed through the `path_registry` table. **Collection URLs**: single collections share the root-URL namespace with posts (`/{slug}` — uniqueness enforced by `path_registry`). Only `/collections` itself (the directory), aggregate selections (`/collections/{a+b}`), and management routes (`/collections/new`, `/collections/{slug}/edit`) live under `/collections`. `SLUG_ID_LENGTH` env var controls random slug length (default: 5).
- **Soft delete**: posts use `deleted_at`.
- **Library functions**: include JSDoc with `@param`, `@returns`, `@example`.

### i18n

All user-facing strings use Lingui message descriptors with a `@context:` comment for translators. In components, import `msg` from `@lingui/core/macro`, get `i18n` from the local context hook, and translate with `i18n._(...)`:

```tsx
import { msg } from "@lingui/core/macro";
import { useLingui } from "../../i18n/context.js";

const { i18n } = useLingui();
return (
  <h1>
    {i18n._(
      msg({
        message: "Settings",
        comment: "@context: Page title",
      }),
    )}
  </h1>
);
```

When interpolating runtime values, keep them in `values`:

```tsx
i18n._(
  msg({
    message: "Found {count} results",
    comment: "@context: Search results count",
  }),
  { count },
);
```

### Tech Stack

Cloudflare Workers, Hono v4, Vite + SWC, Tailwind v4 + BaseCoat, D1 + Drizzle ORM, better-auth, @lingui/core, Datastar v1.0.0-RC.7 (vendored — version matters, APIs vary between releases), Lit (Web Components), Zod, ESLint + Prettier

## UX Copy Guidelines

All user-facing text follows a consistent voice. When writing or reviewing copy, apply these rules.

### Voice & Tone

- Write like a knowledgeable friend — warm but not cute, confident but not arrogant.
- Use plain English. If you can say it in 5 words, don't use 10.
- Avoid filler words: "please", "simply", "just", "easily", "feel free to".
- Never say "successfully" — if it worked, the user knows.

### Empty States

- Never show "No [thing] found" or "No [thing] yet" alone. Always pair it with a next action or brief reason the space is empty.
- Good: `"Nothing published yet. Write your first post to get started."`
- Bad: `"No posts."`

### Buttons & CTAs

- Verb-first, action-specific labels. Avoid generic text.
- Good: `"Publish"`, `"Write your first post"`, `"Delete Media"`
- Bad: `"Submit"`, `"OK"`, `"Confirm"`

### Error Messages

- Always tell the user (1) what went wrong, (2) what they can do about it. Never blame the user.
- Good: `"Wrong email or password. Check your credentials and try again."`
- Bad: `"Invalid input"`

### Success & Confirmation

- Acknowledge the action briefly. Don't say "Success" or "[action] successfully."
- Good: `"Post published."`, `"Settings updated."`, `"Password changed."`
- Bad: `"Operation completed successfully."`

### Destructive Actions

- Be specific about what will be lost. Make it irreversible-sounding if it is.
- Good: `"Delete this post permanently? This can't be undone."`
- Bad: `"Are you sure?"`

### Settings Descriptions

- Describe what a setting **does**, not what it **is**.
- Good: `"Hide this post from search engines and RSS feeds"`
- Bad: `"Visibility: Private"`

## Reference

If you notice code contradicting this document, think about which side is correct, then update whichever is wrong.

### Common Pitfalls

- Combining `.btn` or `.badge` with variant classes (`.btn-outline`, `.btn-ghost`, `.badge-outline`, etc.) — BaseCoat variants are self-contained and combining produces broken styles.
- Importing `useLingui` from `@lingui/react/macro` or writing raw `t({ ... })` calls — in this codebase, prefer `msg(...)` from `@lingui/core/macro` plus `useLingui` from the local i18n context and call `i18n._(msg(...), values?)`.
- Editing `packages/create-jant/template/` — this is auto-generated and will be overwritten.
- Putting multi-service orchestration in route handlers — if two routes need the same sequence of service calls, extract it into a service method. Routes should be thin adapters: parse request → call service → format response.
- Passing plain text to `dangerouslySetInnerHTML` without `escapeHtml()` — even single-author content can contain `<`, `>`, `&`. The pattern for safe highlighted output: `escapeHtml(text).replace(/\x02/g, '<mark>').replace(/\x03/g, '</mark>')` where `char(2)`/`char(3)` (STX/ETX) are used as FTS5 snippet markers in SQL.
- Interpolating runtime values directly into a Lingui `message` template literal — this breaks extraction, makes translations drift, and often hides empty-string / `0` bugs. Use `message: "Found {count} results"` with `values: { count }`, and normalize blank labels before passing them into `values`.

### Docs Index

- **Coding standards** (module deps, error handling, testing): `docs/internal/coding-standards.md`
- **Lit/Datastar conventions**: `docs/internal/lit-guide.md`
- **Testing guide**: `docs/internal/testing-guide.md`
- **Agent automation testing**: `docs/internal/agent-automation-testing.md`
- **Datastar patterns and API**: `docs/datastar.md`, `references/datastar/`
- **BaseCoat components**: `references/basecoat/`
- **Configuration**: `docs/configuration.md`
- **Theming (user-facing)**: `docs/theming.md`
- **Theming (internal design guide)**: `docs/internal/theming.md`
- **Releasing**: `docs/RELEASING.md`
- **Developer onboarding**: `README.md`, `mise tasks`

## Core Principles

– Simplicity First: Make every change as simple as possible
– No Laziness: Find root causes. No temporary fixes
– Minimal Impact: Only touch what’s necessary

---
> Source: [jant-me/jant](https://github.com/jant-me/jant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
