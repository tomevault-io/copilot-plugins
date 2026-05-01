## cattopic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CattoPic is an image hosting service with a Next.js frontend (deployed on Vercel) and a Cloudflare Worker backend (Hono framework) using R2 for storage and D1 for metadata.

**Important: This project uses pnpm as the package manager. Do not use npm or yarn.**

## Commands

### Frontend (Next.js)
```bash
pnpm dev          # Start dev server at localhost:3000
pnpm build        # Build for production
pnpm lint         # Run ESLint
```

### Worker (Cloudflare)
```bash
cd worker
pnpm dev          # Start local worker at localhost:8787
pnpm deploy       # Deploy to Cloudflare
pnpm wrangler d1 execute CattoPic-D1 --remote --file=schema.sql  # Init DB schema
```

## Architecture

```
├── app/                    # Next.js 16 frontend (App Router)
│   ├── components/         # React components
│   ├── hooks/              # Custom React hooks
│   ├── utils/              # Frontend utilities
│   └── manage/             # Admin management page
│
└── worker/                 # Cloudflare Worker backend
    └── src/
        ├── index.ts        # Hono router, routes & middleware
        ├── handlers/       # API endpoint handlers
        │   ├── upload.ts   # Image upload with compression
        │   ├── images.ts   # CRUD operations
        │   ├── random.ts   # Public random image API
        │   └── tags.ts     # Tag management
        └── services/
            ├── storage.ts      # R2 storage operations
            ├── metadata.ts     # D1 database queries
            ├── compression.ts  # Cloudflare Images compression
            └── auth.ts         # API key validation
```

## Key Patterns

### Image Storage Structure
Images are stored in R2 with orientation-based paths:
- `original/{orientation}/{id}.{ext}` - Original file
- `{orientation}/webp/{id}.webp` - WebP compressed
- `{orientation}/avif/{id}.avif` - AVIF compressed

GIF files are stored only as originals (no conversion).

### API Authentication
Protected endpoints require `Authorization: Bearer <api-key>` header. Public endpoints: `/api/random`, `/r2/*`.

### Random Image API
`GET /api/random` defaults to auto-orientation based on User-Agent (mobile→portrait, desktop→landscape). Supports `?orientation=landscape|portrait|auto`, `?tags=`, `?exclude=`, `?format=`.

## Environment Variables

### Frontend (.env.local)
```
NEXT_PUBLIC_WORKER_URL=http://localhost:8787  # or production Worker URL
```

### Worker (wrangler.toml)
Bindings: `R2_BUCKET` (R2), `DB` (D1), `CACHE_KV` (KV), `IMAGES` (Cloudflare Images for compression), `DELETE_QUEUE` (optional, Cloudflare Queue)

Variables: `ENVIRONMENT`, `R2_PUBLIC_URL`, `USE_QUEUE` (`'true'` to enable async Queue-based R2 deletion, `'false'` or omit for synchronous deletion)

## Changelog

**Important: When making functional changes to the codebase, you MUST update the changelog files.**

- `CHANGELOG.md` - English version
- `CHANGELOG_CN.md` - Chinese version

Follow [Keep a Changelog](https://keepachangelog.com/) format with these sections:
- `Added` - New features
- `Changed` - Changes to existing functionality
- `Deprecated` - Features to be removed
- `Removed` - Removed features
- `Fixed` - Bug fixes
- `Security` - Security fixes

---
> Source: [Yuri-NagaSaki/CattoPic](https://github.com/Yuri-NagaSaki/CattoPic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
