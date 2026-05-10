## t2t-bilibili-comments

> Web app displaying translated Bilibili video comments for Ludwig's "Tip to Tip" motorcycle series across China. Translates Chinese audience reactions into English with cultural annotations so international viewers can follow along.

# t2t-bili-comments

Web app displaying translated Bilibili video comments for Ludwig's "Tip to Tip" motorcycle series across China. Translates Chinese audience reactions into English with cultural annotations so international viewers can follow along.

**Stack:** Next.js 16 (App Router), React 19, Tailwind CSS 4, Radix UI, Bun, Drizzle ORM, Supabase (Postgres), Playwright, Claude API, oxlint, oxfmt
**Deploy:** Vercel, comment JSON on Cloudflare R2, images on R2, metadata in Supabase

## Critical Constraints

**Annotation marker/array parity:** Comments use `[~N:translated phrase]` markers in content that reference an `annotations[]` array on the comment object. Every marker MUST have a matching annotation entry (by index), and every annotation MUST have a marker in the content. Mismatches break the tooltip rendering in `AnnotatedText.tsx`. Validation logic lives in `lib/annotations.ts`. The translation prompt rules live in `tools/prompt-template.md`.

**Emote preservation:** Emote codes like `[笑哭]` must be preserved verbatim in translations. They are rendered as inline images via `emoteMap` lookup in `RichText.tsx`.

**Data architecture — Supabase metadata + R2 blobs:** Comment JSON blobs live in R2 (cheap storage). Supabase stores metadata only: R2 keys, comment counts, timestamps, categories, videos. `lib/comments.ts` queries Supabase for R2 keys, then fetches the JSON blob from R2. Overrides (approved edits) are stored in Supabase and merged at read time.

**Provider-agnostic schema:** The `videos` table uses a generic `id` + `provider` enum (`bilibili | youtube | other`) instead of Bilibili-specific fields. This supports future non-Bilibili sources.

**Server-only modules:** `lib/comments.ts` and `lib/config.ts` query Supabase — never import them from client components. All data loading happens server-side.

**URL rewriting:** `lib/comments.ts` rewrites Bilibili CDN URLs (`i0-i2.hdslb.com`) to `NEXT_PUBLIC_ASSETS_URL` at load time. For `new_dyn/` picture URLs, translated variants use `.en.jpg` suffix. Don't hardcode Bilibili CDN URLs in components.

**Translation resume:** `tools/translate.ts` tracks `translatedIds` in file metadata to skip already-translated comments on re-run. Don't break this field.

**Drizzle migrations:** Never manually edit files in `drizzle/` (migrations, journal, snapshots). Always use `bunx drizzle-kit generate` then `bunx drizzle-kit migrate`. Schema source of truth is `lib/db/schema.ts`.

**UI primitives — Radix UI:** Use `radix-ui` for dialogs, popovers, tooltips, and other interactive primitives. Import as `import { Dialog, Popover, Tooltip } from "radix-ui"` and use compound components (`Dialog.Root`, `Dialog.Content`, etc.). Each component manages its own open/close state locally — no global overlay store. Radix handles scroll lock, portals, focus trapping, and accessibility.

## Architecture & Data Flow

### Comment Pipeline

```
fetch-comments (Playwright + Bilibili API)
  -> comments/{categoryId}/{videoId}.json (local)
  -> translate (Claude Code CLI, batched)
  -> comments/{categoryId}/{videoId}.en.json (local)
  -> download-assets (Bilibili CDN images to .assets/)
  -> upload-assets:
     Phase 1: images to R2
     Phase 2: comment JSON to R2 → upsert comment_data + insert fetch_snapshots in Supabase
     Phase 3: danmaku XML to R2
```

### Server-Side Data Loading

```
Request hits Vercel
  -> lib/config.ts queries Supabase for categories + videos
  -> lib/comments.ts queries Supabase for R2 keys
  -> fetches JSON blobs from R2
  -> queries overrides table, merges approved edits
  -> rewriteCommentUrls() rewrites Bilibili CDN URLs
  -> Page renders with full data (no client-side fetching)
```

### Edit Flow

```
User submits edit → POST /api/edits (validates annotations)
  -> edits table (status: pending)
Admin approves → POST /api/admin/edits/[id]/approve
  -> overrides table (upsert)
```

### Danmaku Pipeline

```
Bilibili danmaku XML
  -> translate-danmaku (Claude)
  -> merge-danmaku (per-category merge)
  -> danmaku/{categoryId}.xml + danmaku/{categoryId}.en.xml
  -> danmaku/manifest.json (on R2)
```

### Database Schema

- `categories` — id, title, description, maxAge, sortOrder
- `videos` — id (provider-specific), provider enum, categoryId, title, url, sortOrder
- `comment_data` — videoId, lang, r2Key, commentCount, fetchedAt, translatedAt (current state)
- `fetch_snapshots` — videoId, lang, r2Key, commentCount, viewCount, likeCount, danmakuCount, fetchedAt (time-series)
- `edits` — videoId, commentId, content, annotations, status (pending/approved/rejected)
- `overrides` — videoId, commentId, content, annotations, editId (approved edits applied at read time)

### Directory Layout

- `app/` — Next.js App Router pages + API routes
- `app/login/` — Supabase auth login page
- `app/auth/callback/` — OAuth/email verification callback
- `app/admin/edits/` — Admin edit review dashboard
- `app/api/edits/` — Edit submission endpoint
- `app/api/admin/edits/` — Admin approve/reject endpoints
- `components/` — React components organized by domain. Client components use `"use client"`
  - `comments/` — Comment, CommentThread, CommentPictures, EditModal, AnnotationEditor, AnnotatedText, RichText
  - `layout/` — AppShell, CategorySidebar, CategoryHeader, HomeHeader, AuthButton, ThemeProvider
  - `category/` — CategoryCommentsView, CategoryViewContext, VideoCard, VideoStatsBar
  - `controls/` — SortControls, LanguageToggle, UnreadBadge
  - `ui/` — ThemeToggle, SegmentedControl, Switch, TimeAgo, QueryProvider
  - `danmaku/` — EmojiGlossary, DanmakuDownload
- `lib/` — Shared utilities and data access
  - `types.ts`, `config.ts`, `comments.ts`, `annotations.ts`, `store.ts`, `auth.ts`, `env.ts`, `logger.ts`
  - `db/` — Drizzle ORM: `schema.ts` (table definitions), `index.ts` (client)
  - `supabase/` — `client.ts` (browser client), `server.ts` (SSR client)
  - `utils/` — `date.ts`, `format.ts`, `emote-glossary.ts`, `fetch-with-timeout.ts`
- `hooks/` — Custom hooks (`useUnread.ts`)
- `tools/` — CLI scripts for the data pipeline
- `comments/` — Committed JSON data (local copies, uploaded to R2)
- `danmaku/` — Merged danmaku XML files
- `drizzle/` — Generated migration files (do not edit manually)

## Key Conventions

**Colors:** Bilibili-themed. Use Tailwind classes `bili-pink` (#fb7299) for likes/accents and `bili-blue` (#00a1d6) for primary actions/links. Defined in `app/globals.css` as CSS custom properties.

**Responsive:** Mobile-first. Base styles target mobile; use `md:` for tablets (e.g., side-by-side comments grid) and `lg:` for desktop (e.g., persistent sidebar).

**UI state:** Persistent UI state (sort mode, language view, annotation toggle) is managed via URL search params (`?sort=hot&lang=side-by-side&ann=1`), not React state.

**Types:** All TypeScript interfaces in `lib/types.ts`. DB schema in `lib/db/schema.ts`.

**Config:** Categories and videos are stored in Supabase (`categories` + `videos` tables). `lib/config.ts` provides `getCategory(id)`, `getAllCategorySlugs()`, `getAllCategoriesWithSummary()` — all async, wrapped with `React.cache()`.

**Package manager:** Bun for everything (`bun install`, `bun run dev`, `bun run tools/...`). Exception: `fetch-comments` uses `npx tsx` because it needs Playwright which requires Node.

## Scripts

| Command                    | Purpose                                                   |
| -------------------------- | --------------------------------------------------------- |
| `bun run dev`              | Next.js dev server                                        |
| `bun run build`            | Next.js production build                                  |
| `bun run lint`             | oxlint                                                    |
| `bun run lint:fix`         | oxlint with auto-fix                                      |
| `bun run format`           | oxfmt (format code)                                       |
| `bun run format:check`     | oxfmt (check formatting)                                  |
| `bun run knip`             | Find unused exports, files, and dependencies              |
| `bun run fetch-comments`   | Fetch comments from Bilibili via Playwright               |
| `bun run translate <file>` | Translate a comment JSON file via Claude CLI              |
| `bun run download-assets`  | Download referenced Bilibili CDN images to `.assets/`     |
| `bun run upload-assets`    | Upload images + comments to R2, sync metadata to Supabase |
| `bun run update-all`       | Full pipeline: fetch + translate + download + upload      |
| `bun run tools/seed-db.ts` | Seed Supabase from local JSON files                       |

## Key Files

| File                                            | Purpose                                                           |
| ----------------------------------------------- | ----------------------------------------------------------------- |
| `lib/db/schema.ts`                              | Drizzle ORM table definitions (source of truth for DB schema)     |
| `lib/db/index.ts`                               | Drizzle client (connects to Supabase)                             |
| `lib/types.ts`                                  | All TypeScript interfaces                                         |
| `lib/comments.ts`                               | Comment loading: DB query for R2 keys → R2 fetch → override merge |
| `lib/config.ts`                                 | Category/video queries from Supabase                              |
| `lib/annotations.ts`                            | Annotation marker/array parity validation                         |
| `lib/store.ts`                                  | Zustand store: auth state, drawer, emote glossary                 |
| `lib/auth.ts`                                   | `requireAuth()` + `requireAdmin()` server-side guards             |
| `components/comments/AnnotationEditor.tsx`      | Rich contenteditable editor with inline annotation management     |
| `components/comments/EditModal.tsx`             | Edit submission dialog (Radix Dialog)                             |
| `components/comments/CommentPictures.tsx`       | Thumbnail grid + lightbox (Radix Dialog)                          |
| `components/comments/AnnotatedText.tsx`         | Annotation tooltip on hover (Radix Tooltip)                       |
| `app/api/edits/route.ts`                        | Edit submission endpoint                                          |
| `app/api/admin/edits/[editId]/approve/route.ts` | Admin edit approval                                               |
| `app/api/admin/edits/[editId]/reject/route.ts`  | Admin edit rejection                                              |
| `app/login/page.tsx`                            | Supabase email/password login                                     |
| `app/admin/edits/page.tsx`                      | Admin edit review dashboard                                       |
| `tools/translate.ts`                            | Batch translation script with resume support                      |
| `tools/upload-assets.ts`                        | R2 upload + Supabase metadata sync                                |
| `tools/seed-db.ts`                              | Database seeder from local JSON files                             |

## Environment Variables

| Variable                 | Purpose                                     |
| ------------------------ | ------------------------------------------- |
| `DATABASE_URL`           | Supabase pooled connection string           |
| `DATABASE_DIRECT_URL`    | Supabase direct connection (for migrations) |
| `NEXT_PUBLIC_ASSETS_URL` | Base URL for R2 CDN (required)              |
| `ADMIN_SECRET`           | Shared secret for admin API routes          |
| `S3_ENDPOINT`            | R2 endpoint for uploads                     |
| `S3_ACCESS_KEY_ID`       | R2 access key                               |
| `S3_SECRET_ACCESS_KEY`   | R2 secret key                               |
| `S3_BUCKET_NAME`         | R2 bucket name                              |

---
> Source: [capybara-baron/t2t-bilibili-comments](https://github.com/capybara-baron/t2t-bilibili-comments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
