## rjk-website

> This installs dependencies and registers the pre-commit hook that runs `pnpm test` before every commit.

# CLAUDE.md - Project Guidelines for Claude Code

## Setup (first-run after cloning)
```bash
./dev_setup.sh
```
This installs dependencies and registers the pre-commit hook that runs `pnpm test` before every commit.

## Project Overview
Artist portfolio website for Rebecca Kleinberg built with Next.js 16, React 19, TypeScript, Tailwind CSS, and shadcn/ui. Content (artwork, blog posts) is stored in **Notion** and fetched via the Notion API. Images are downloaded from Notion at build time into `public/images/` (git-ignored).

## Environment Variables
Copy `.env.example` to `.env.local` and fill in the Notion credentials before running locally.

## Commands
- `pnpm test` - Run all tests once
- `pnpm test:watch` - Run tests in watch mode
- `pnpm test:coverage` - Run tests with coverage report
- `pnpm dev` - Start development server
- `pnpm build` - Production build (downloads Notion images via `scripts/`)
- `pnpm lint` - Run ESLint

## Architecture
- **App Router** (Next.js 16): Pages in `app/`, components in `components/`, utilities in `lib/`, hooks in `hooks/`
- **Path alias**: `@/*` maps to project root
- **Pages**: `app/page.tsx` (home), `app/about/`, `app/work/paintings/`, `app/work/drawings/`, `app/blog/` (listing + `[slug]` dynamic route), `app/rebuild/` (password-protected rebuild trigger UI, `.vercel.app`-only)
- **UI components**: `components/ui/` contains shadcn/ui primitives -- DO NOT modify or test these
- **Custom components**: `components/gallery-grid.tsx`, `components/lightbox.tsx`, `components/navigation.tsx`
- **Notion layer** (`lib/`):
  - `notion.ts` — Notion API client; fetch functions for artworks and blog posts
  - `notion-renderer.tsx` — React renderer for Notion block content
  - `types.ts` — Shared TypeScript interfaces (`Artwork`, `BlogPost`, `NotionBlock`, etc.)
  - `artwork-data.ts` — Type re-exports only (data now lives in Notion)
- **Build scripts** (`scripts/`): `download-notion-images.ts`, `download-blog-images.ts` download images at build time; one-off utilities also live here
- **CI/CD**: `.github/workflows/` — Vercel preview/production deploy and content-rebuild workflows
- **Rebuild API** (`app/api/rebuild/`):
  - `_auth.ts` — shared secret validation helper
  - `staging/route.ts` — fires `notion-content-staging` repository_dispatch to trigger a preview build
  - `promote/route.ts` — fires `notion-content-promote` repository_dispatch to trigger a fresh production build and deploy
  - Both endpoints accept `?secret=` query param or `Authorization: Bearer` header checked against `REBUILD_SECRET`
- **Middleware** (`middleware.ts`): Restricts `/rebuild` to `.vercel.app` domains only (blocks the page on production custom domains)
- **Public folder**: `public/images/` dirs are git-ignored (populated at build time). Tests MUST NOT rely on files in `public/`

## Testing Rules

### MANDATORY: Tests Must Accompany All Changes
1. **Every new feature** must include corresponding tests before the change is considered complete
2. **Every bug fix** must include a regression test that fails without the fix
3. **Every modified file** requires updating its existing tests if behavior changed
4. **Run `pnpm test` after creating a commit** — Claude Code will automatically run tests to verify the commit

### Automatic Test Running
Claude Code is configured to run `pnpm test` after each commit you create. If tests fail, the commit will still exist but you should fix the failing tests immediately and create a follow-up commit.

### Test Framework
- **Vitest** with jsdom environment and React Testing Library
- Test files live in `__tests__/` directories colocated with source files
- Test file naming: `{source-name}.test.ts` or `{source-name}.test.tsx`

### Test Structure by Layer
- **`lib/__tests__/`**: Pure unit tests, no mocks needed
- **`hooks/__tests__/`**: Use `renderHook`, mock browser APIs (matchMedia, timers)
- **`components/__tests__/`**: Use `render` + user-event, mock next/image, next/link, next/navigation
- **`app/**/__tests__/`**: Page smoke tests verifying key content renders

### Image Mocking (Critical)
- `next/image` is globally mocked in `vitest.setup.tsx` to render plain `<img>` tags
- `next/link` is globally mocked in `vitest.setup.tsx` to render plain `<a>` tags
- Use `createMockArtwork()` from `__tests__/helpers/mock-artwork.ts` for test data
- NEVER reference actual files from `public/` in tests

### What NOT to Test
- shadcn/ui components in `components/ui/` (third-party)
- CSS/Tailwind class names (visual aspects)
- next.config.mjs, tailwind.config.ts, postcss.config.mjs (config files)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ja-ba)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ja-ba)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
