## exif-gallery-nuxt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## User Preferences

**Important**: The user prefers communication in Chinese (中文). When working on this project:

- Keep conversations in Chinese
- Use Context7 MCP for official documentation lookups when encountering complex/uncertain issues
- Never use `any` type in TypeScript code - maintain strict type safety
- Do not modify files listed in `.gitignore`
- Always run `pnpm lint --fix` after making code changes

## Project Overview

EXIF Gallery Nuxt is a full-stack photo gallery solution deployable on Cloudflare Workers. It features AI-powered image analysis (OpenAI/Gemini), browser-side image compression (JSQuash), complete EXIF metadata management, and edge-native storage using Cloudflare R2 and D1.

**Tech Stack**: Nuxt 4 + Vue 3.5 + NuxtHub + Cloudflare (D1/R2) + UnoCSS + shadcn-vue + Pinia + Drizzle ORM

## Development Commands

```bash
# Development
pnpm dev                    # Start dev server (localhost:3000)
pnpm dev --remote          # Connect to remote Cloudflare resources locally

# Build & Deploy
pnpm build                 # Production build
pnpm preview               # Preview production build
pnpm deploy                # Build and deploy to Cloudflare Workers

# Database
pnpm db:generate           # Generate Drizzle migrations from schema changes

# Code Quality
pnpm lint --fix            # ESLint with @antfu/eslint-config
pnpm typecheck             # Vue + TypeScript type checking

# UI Components
pnpm ui add <component>    # Add shadcn-vue component

# Logs (post-deployment)
pnpm logs                  # Production deployment logs
pnpm logs:preview          # Preview environment logs
```

## Architecture Overview

### Edge-First Data Flow

The application follows a Cloudflare-native architecture where all data lives at the edge:

1. **Upload Flow** (Browser → Server → R2):
   - Browser: Image selected → EXIF extracted via `exifr` → Compressed via JSQuash Web Workers (`app/workers/encode.worker.ts`)
   - Multiple formats generated: original JPEG + optimized WebP + modern AVIF + thumbnail
   - Auto-resize: images with short edge ≥ 2880px are scaled down to 2160px while preserving aspect ratio
   - Server API (`server/api/photos/upload.post.ts`): Receives base64 blobs → Uploads to R2 via NuxtHub → Stores metadata in D1
   - Optional AI analysis: Compressed image sent to OpenAI/Gemini to generate `title`, `caption`, `tags`, `semanticDescription`

2. **Storage Layer**:
   - **D1 (SQLite)**: Photo metadata and relationships (EXIF, tags, associations)
   - **R2 (S3-compatible)**: Binary image blobs served via `/photos/[pathname]` route with aggressive caching (`Cache-Control: public, max-age=31536000, immutable`)
   - Schema: `photos` (main), `tags` (normalized), `photo_tags` (many-to-many junction table)

3. **Query & Display Flow**:
   - API (`server/api/photos/index.get.ts`): Supports pagination (`limit`/`offset`), filtering (by `tag`/`camera`/`lens`/`hidden`), sorting (`takenAt`/`createdAt`)
   - Composable (`app/composables/usePhotos.ts`): `usePhotosInfinite` for infinite scroll, `usePhoto` for single item
   - Pinia store (`app/stores/photos.ts`): Client-side cache with infinite scroll state management

### Database Schema Relationships

The schema uses a modern normalized tag system (migrated from legacy comma-separated `photos.tags` field):

```
photos (1) ←→ (N) photo_tags (N) ←→ (1) tags
```

- `photos.id` (CUID, 8 chars): Primary key for photo records
- `tags.name` (unique): Canonical tag names with `photoCount` denormalized counter
- `photo_tags`: Junction table with cascade delete on both sides
- Indexes: `idx_photos_taken_at`, `idx_photos_hidden`, `idx_tags_photo_count`, `idx_photo_tags_photo_id`, `idx_photo_tags_tag_id`

### AI Integration Architecture

Multi-provider AI configuration managed client-side via localStorage (`app/composables/useAIConfig.ts`):

- Supports OpenAI and Gemini with custom base URL overrides (useful for proxy services)
- Uses `ai-sdk`'s `generateObject` with Zod schema for type-safe structured output
- Image compression before AI analysis: reduces token costs and respects API size limits
- Output: `{ title: string, caption: string, tags: string[], semanticDescription: string }`

### Browser-Side Image Processing

Image compression runs entirely in the browser via Web Workers to avoid server load:

1. **Workers** (`app/workers/`):
   - `decode.worker.ts`: Decodes uploaded images to raw pixel data
   - `encode.worker.ts`: Encodes pixel data to JPEG/WebP/AVIF formats with quality settings

2. **Compression Pipeline** (`app/utils/compress.ts`):
   - Reads file → Extracts EXIF (before compression destroys metadata) → Spawns workers
   - Auto-resize logic: `Math.min(width, height) >= 2880 ? resize to 2160 : keep original`
   - Quality presets: JPEG 0.85, WebP 0.85, AVIF 0.65, Thumbnail 0.7 at 400px

3. **Configuration** (`app/composables/useUploadConfig.ts`):
   - User-controllable: toggle compression, enable/disable specific formats
   - Stored in localStorage, persists across sessions

### Route Patterns & Page Organization

- `/` (index.vue): Home page with featured photos
- `/grid`: Grid view of all photos
- `/p/[...id]`: Photo detail page with EXIF overlay and viewer.js lightbox
- `/tag/[...tag]`, `/camera/[...camera]`, `/lens/[...lens]`: Filtered views
- `/admin/*`: Protected routes with `auth` middleware, requires `NUXT_ADMIN_PASSWORD`
- Layouts: `default.vue`, `home.vue`, `admin.vue`

### Authentication & Security

- Session-based auth via `nuxt-auth-utils`
- Admin login: POST `/api/auth` with password matching `NUXT_ADMIN_PASSWORD` env var
- Session encryption: `NUXT_SESSION_PASSWORD` (min 32 chars)
- Protected routes use `definePageMeta({ middleware: 'auth' })` or `requireUserSession(event)` on API routes

## Migration Management (Critical)

Cloudflare D1 cannot be connected during build time, so migrations are **not** auto-applied. Two management strategies:

1. **Local Development**: NuxtHub auto-manages migrations, records in `_hub_migrations` table (no `.sql` suffix)
2. **Cloud Deployment**: GitHub Actions workflow (`.github/workflows/migrate.yml`) runs `wrangler d1 migrations apply`, records with `.sql` suffix

**Important**: Never manually run `wrangler d1 migrations` commands during local dev - suffix mismatch will cause duplicate migration tracking.

When modifying schema:

1. Edit `server/db/schema.ts`
2. Run `pnpm db:generate` to create migration in `server/db/migrations/sqlite/`
3. Commit migration file
4. Push to `main` branch → GitHub Actions auto-applies to production D1

## Component Structure

- `app/components/ui/`: shadcn-vue base components (Button, Dialog, Card, etc.)
- `app/components/inspira/`: inspira-ui animated components (3D effects, motions)
- `app/components/ui-pro/`: Project-specific extended UI components
- Auto-import enabled for all components, composables, and utils
- Use `<script setup lang="ts">` with Composition API exclusively

## Code Style

- **ESLint**: `@antfu/eslint-config` (single quotes, no semicolons, strict TypeScript)
- **Prohibited**: `any` type usage (user's explicit rule)
- **i18n**: Use `$t('key')` in templates, translation files in `i18n/locales/` (en.yml, zh.yml)
- **Type Safety**: Runtime validation with Zod, compile-time with TypeScript strict mode

## Environment Variables

Required:

- `NUXT_ADMIN_PASSWORD`: Admin panel password (default: `admin`)
- `NUXT_SESSION_PASSWORD`: Session encryption key (min 32 chars, no default)

Optional:

- `NUXT_PUBLIC_TITLE`: Application title (default: "Exif Gallery Nuxt")
- `NUXT_PUBLIC_DESCRIPTION`: Meta description
- `NUXT_PUBLIC_DISABLE_3D_CARD_DEFAULT`: Disable 3D card effects (`"true"`/`"false"`)

## Key Files for Common Tasks

**Adding photo metadata fields**:

1. `server/db/schema.ts`: Add column to `photo` table
2. `pnpm db:generate`: Generate migration
3. `server/api/photos/upload.post.ts`: Handle new field in upload logic
4. `app/composables/usePhotos.ts`: Update type definitions
5. UI components displaying photo info

**Adding API endpoints**:

- Create `server/api/[name]/[method].ts` (e.g., `index.get.ts`, `[id].put.ts`)
- Use `eventHandler()` wrapper
- Access DB: `useDB()` from NuxtHub
- Require auth: `await requireUserSession(event)`

**Adding AI providers**:

- `app/utils/aiProviders.ts`: Define provider schema and defaults
- `app/composables/useAIConfig.ts`: Provider CRUD logic already implemented
- UI: Admin panel already supports custom provider configuration

## Deployment Configuration

`wrangler.jsonc` binds Cloudflare resources:

```jsonc
{
  "d1_databases": [{ "binding": "DB", "database_id": "xxx" }],
  "r2_buckets": [{ "binding": "BLOB", "bucket_name": "xxx" }]
}
```

GitHub Actions secrets required:

- `CLOUDFLARE_ACCOUNT_ID`: From Cloudflare dashboard
- `CLOUDFLARE_API_TOKEN`: With D1 edit permissions

---
> Source: [wiidede/exif-gallery-nuxt](https://github.com/wiidede/exif-gallery-nuxt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
