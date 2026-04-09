## 2026-ffx-summer-camps

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
npm install

# Run development server (port 3000)
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Convert XLSX camp data to JSON (with geocoding)
bun scripts/xlsx-to-json.ts data/FCPA\ Camp\ Spreadsheet.xlsx data/fcpa-camps.json
```

## Architecture

This is a Fairfax County Parks summer camp explorer built with Next.js 15 (App Router).

### Data Flow
1. **Source**: `data/FCPA Camp Spreadsheet.xlsx` - raw camp data from FCPA
2. **Conversion**: `scripts/xlsx-to-json.ts` parses the XLSX, infers types (dates, times, ages, fees), geocodes locations via Nominatim (cached in `data/.geocode-cache.json`), and outputs structured JSON
3. **API**: `/app/api/camps/route.ts` serves `data/fcpa-camps.json` at `/api/camps`
4. **Frontend**: `/app/page.tsx` is a client component that fetches and renders camps with filtering

### Key Files
- `app/layout.tsx` - Root layout with metadata, fonts, and Vercel Analytics
- `app/page.tsx` - Main client component with all camp explorer UI (~2000 lines)
- `app/globals.css` - Tailwind CSS and custom styles
- `app/api/camps/route.ts` - API route serving camp data
- `scripts/xlsx-to-json.ts` - Data pipeline with geocoding and type inference
- `data/location-addresses.json` - Manual address mappings for geocoding failures

### Frontend State
- Filter state persists to URL params (`q`, `cat`, `comm`, `loc`, `week`, `minAge`, `maxAge`, `maxFee`, `view`)
- Map uses Leaflet loaded dynamically from CDN
- Custom marker icons rendered as inline SVG
- Favorites and user location stored in localStorage

## Next.js Specifics

- Uses App Router with Turbopack for development
- The main page is a client component (`"use client"`) due to browser API usage
- API routes use the new Route Handlers format
- Tailwind CSS configured via `tailwind.config.ts`
- Static assets go in `/public`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dakridge)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dakridge)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
