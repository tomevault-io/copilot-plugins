## ilia-beer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ilia.beer** is a "Buy Me a Beer" support platform with a Next.js frontend and PHP backend. Users can upload video messages that are stored in Google Cloud Storage.

## Development Commands

### Frontend (beerfront/)
```bash
cd beerfront
npm run dev      # Start development server
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # Run ESLint
```

### Backend (backend/)
```bash
cd backend
docker-compose up -d              # Start all services (PHP-FPM, Nginx, MySQL)
docker-compose down               # Stop services
docker-compose logs -f backend    # View backend logs
```

## Architecture

### Monorepo Structure
- **beerfront/** - Next.js 19 frontend (React 19, TypeScript, Tailwind CSS 4)
- **backend/** - PHP 8.2 REST API with MySQL and Google Cloud integration

### Frontend Architecture
- **App Router**: Routes in `app/` with `page.tsx` files
- **API Layer**: `app/lib/api/client.ts` (Axios configured for `https://api.ilia.beer`), `app/lib/api/videos.ts` (video service)
- **UI Components**: `app/ui/` contains video-card, mesh-gradient-background, etc.
- **Providers**: `app/providers/lenis-provider.tsx` for smooth scrolling

Key patterns:
- Most components are React Server Components; use `'use client'` sparingly
- Videos preload sequentially via refs to prevent bandwidth spikes
- IntersectionObserver triggers viewport-based video playback
- Thumbnails generated client-side via canvas frame capture

### Backend Architecture
- **Controller**: `src/Controller/VideoController.php` - REST endpoints
- **Gateway**: `src/TableGateways/VideoGateway.php` - PDO data access layer
- **System**: `src/System/DatabaseConnector.php` - MySQL connection
- **Bootstrap**: `bootstrap.php` initializes DB, GCS bucket, and Video Transcoder clients

API endpoints:
- `GET /api/videos` - List all videos
- `POST /api/videos/signed-url` - Get GCS upload URL
- `PUT /api/videos/{id}` - Update video caption
- `DELETE /api/videos/{id}` - Delete video

### Infrastructure
- **GCP Project**: `facilitypipeline`
- **GCS Bucket**: `ilia_beer`
- Nginx reverse proxy to PHP-FPM on port 9000
- Cloudflare tunnel available via `docker-compose.tunnel.yml`

## Environment Variables

### Frontend (.env.local)
```
NEXT_PUBLIC_BACKEND_URL=https://api.ilia.beer
```

### Backend (.env)
```
DB_HOST, DB_PORT, DB_DATABASE, DB_USERNAME, DB_PASSWORD
GOOGLE_APPLICATION_CREDENTIALS=GCS.json
```

## Code Conventions

### Next.js
- Use kebab-case for directories, PascalCase for component files
- Prefer named exports: `export function Button()` not `export default`
- Minimize `'use client'` - keep most components as RSC
- Wrap interactive client components in `Suspense` with fallback UI

### Tailwind CSS
- Mobile-first with responsive prefixes (`md:`, `lg:`)
- Use CSS variables for theming consistency

### Design Aesthetic
- Dark theme with warm amber/bronze accents
- Fonts: Pacifico (display), Outfit (body), Playfair Display
- Framer Motion for animations with staggered reveals
- Avoid generic fonts (Inter, Roboto) and cliched color schemes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ramzxy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ramzxy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
