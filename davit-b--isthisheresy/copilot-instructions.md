## isthisheresy

> **Is This Heresy?** - A minimalist Next.js infographic site for health information. No headers, footers, or newsletter captures—just the content with AI verification buttons.

# CLAUDE.md

## Project Overview

**Is This Heresy?** - A minimalist Next.js infographic site for health information. No headers, footers, or newsletter captures—just the content with AI verification buttons.

## Tech Stack

- Next.js 14 (App Router)
- TypeScript
- React 18
- Lucide React (icons)
- Sharp (image processing)

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server (localhost:3000)
npm run build        # Production build
npm run convert-images  # Convert PNGs in dev/native-png/ to WebP
```

## Project Structure

```
isthisheresy-next/
├── app/
│   ├── page.tsx              # Homepage with brick mosaic
│   ├── [topic]/page.tsx      # Dynamic infographic pages
│   ├── layout.tsx            # Root layout, fonts, meta
│   └── globals.css           # Global styles
├── components/
│   ├── Brick.tsx             # Homepage brick button
│   ├── LeftRail.tsx          # Left sidebar with topic list
│   ├── InfographicViewer.tsx # Zoomable/pannable image viewer
│   └── BottomBar.tsx         # Verify buttons, share, next preview
├── data/
│   └── topics.ts             # All topic data (single source of truth)
├── dev/
│   └── native-png/           # Source PNGs (gitignored)
├── public/
│   └── images/               # Generated WebP files (committed)
└── scripts/
    └── convert-images.mjs    # PNG → WebP conversion
```

## Data Model

Topics are defined in `data/topics.ts`. Each topic has:

| Field | Description |
|-------|-------------|
| `id` | URL slug (`/glyphosate`) |
| `brickTitle` | One word for homepage (uppercase) |
| `longTitle` | Full title for page and OpenGraph |
| `shareSnippet` | Short description for sharing (<160 chars) |
| `imageName` | Base filename (must match PNG name) |
| `verifyPrompt` | Full prompt for AI fact-checking |

## Adding New Infographics

1. Drop PNG in `dev/native-png/`
2. Run `npm run convert-images`
3. Add entry to `data/topics.ts`

## Code Style

- Inline styles are used throughout (no CSS modules or Tailwind)
- Components are simple functional components
- No state management library—React state only
- Static site—no backend or API routes

## Deployment

Vercel recommended. No environment variables needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davit-b) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
