## image2asset

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Image2Asset is a Next.js 14 web application that converts images to 3D models through a multi-step pipeline using AI services (Google Gemini for image generation, Bria for background removal, and Trellis-2 for 3D conversion via Fal.ai).

## Commands

```bash
npm run dev      # Start development server (port 3000)
npm run build    # Production build
npm run lint     # ESLint via Next.js
npm start        # Start production server
```

## Environment Setup

Copy `.env.example` to `.env.local` and add:
- `GEMINI_API_KEY` - Google Gemini API key
- `FAL_KEY` - Fal.ai API key (used for Bria and Trellis)

## Architecture

### Three-Tier Structure

**Client Layer** (`/app`, `/components`)
- App Router pages: main pipeline (`/app/page.tsx`), gallery (`/app/gallery/page.tsx`)
- `components/pipeline/` - Step wizard components (ImageGenStep, BackgroundRemovalStep, Convert3DStep, BulkProcessor)
- `components/gallery/` - Asset management components
- `components/ui/` - Shared components including ModelViewer (Three.js/React Three Fiber)

**API Layer** (`/app/api`)
- `/api/generate` - Gemini image generation
- `/api/remove-bg` - Bria background removal
- `/api/convert-3d` - Trellis 2D-to-3D conversion
- `/api/enhance` - Gemini image enhancement
- `/api/bulk` - Batch processing with concurrency control
- `/api/jobs` - Job status tracking
- `/api/assets` - Asset CRUD and upload

**Service Layer** (`/lib/services`)
- `BaseService` - Abstract class with retry logic, timeout handling, progress callbacks
- `GeminiService`, `BriaService`, `TrellisService` - Extend BaseService

**Storage Layer** (`/lib/storage`)
- `assets.ts` - Asset registry (`/data/assets.json`), file management
- `jobs.ts` - Job tracking (`/data/jobs.json`) with parent-child relationships

### Data Storage Locations

- `/public/assets/{generated,no-bg,uploads}/` - Image files
- `/models/` - GLB 3D model files
- `/data/` - JSON registries (created at runtime)

### Configuration

`config.json` controls service settings (Gemini model, Trellis texture size/mesh simplification), bulk processing (concurrency, retries), and storage paths. Validated with Zod schemas in `/lib/config/`.

## Key Patterns

- Path alias: `@/*` resolves to project root
- All service calls use exponential backoff retry (configurable attempts)
- Asset lineage tracking: source → no-bg → 3D model relationships
- Progress callbacks for real-time UI updates on async operations
- Local URLs converted to data URLs for Fal.ai API compatibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arisylafeta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
