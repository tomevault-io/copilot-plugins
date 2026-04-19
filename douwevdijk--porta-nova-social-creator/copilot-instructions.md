## porta-nova-social-creator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Porta Nova Social Creator — a React SPA for florists to create social media content featuring Porta Nova Red Naomi roses. Users select an occasion, pick a photo, and generate AI captions via Google Gemini, then preview/download a composited 1080x1080 Instagram-ready image with text overlay.

## Commands

- `npm run dev` — Start dev server on port 3000
- `npm run build` — Production build
- `npm run preview` — Preview production build

No test runner or linter is configured.

## Architecture

**Single-page app with flat file structure.** No router — views toggle via `isAdmin` state.

- **App.tsx** — Monolithic component (~650 lines) containing all UI, state, and canvas rendering logic
- **types.ts** — Enums (`FloralEvent`, `ContentLanguage`, `OverlayPosition`) and interfaces (`RoseImage`, `GeneratedCaption`, `AppState`)
- **constants.ts** — Pre-configured image gallery (20 images hosted at portanova.nl) with event tags
- **services/geminiService.ts** — Gemini AI integration using `@google/genai` SDK; calls `gemini-3-flash-preview` model, returns 3 structured caption variations (title, body, hashtags)
- **index.html** — Entry point; loads Tailwind CSS via CDN, defines import maps for ESM dependencies (React, Gemini SDK, Lucide icons from esm.sh), custom fonts (Playfair Display, Plus Jakarta Sans)

**State management:** React hooks only (useState/useEffect/useMemo). No external state library. localStorage persists custom images (`pn_custom_images_v3`) and tag overrides (`pn_tag_overrides_v3`).

**Styling:** Tailwind CSS via CDN with brand colors as CSS custom properties (`--pn-bronze: #907B4F`, `--pn-red: #8B0000`).

**Image compositing:** Canvas 2D API creates final images — renders photo, gradient overlay, and text at configurable position/size. Uses `document.fonts.ready` for font preloading before canvas render.

## Key Workflows

1. **Content creation flow:** Select event → filter gallery → pick image → generate AI captions → customize overlay → preview → share/download
2. **Admin mode:** Password-protected panel for uploading custom images (stored as base64 in localStorage), managing event tags per image, and exporting config as JSON

## Environment

Requires `GEMINI_API_KEY` in `.env.local`. Vite config exposes it as both `process.env.API_KEY` and `process.env.GEMINI_API_KEY`.

## Path Aliases

`@/*` maps to project root (configured in both tsconfig.json and vite.config.ts).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/douwevdijk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
