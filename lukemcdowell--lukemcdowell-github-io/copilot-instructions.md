## lukemcdowell-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Frontend (Astro)
```bash
npm run dev       # Start dev server
npm run build     # Build static site
npm run preview   # Preview production build
```

### Infrastructure (AWS CDK) — run from `infra/`
```bash
npm run build     # Compile TypeScript
npm run test      # Run Jest tests
npx cdk deploy    # Deploy stack
npx cdk diff      # Preview changes
```

## Architecture

This is a personal portfolio site built with Astro. The frontend is a static site deployed to GitHub Pages; the backend is an AWS Lambda + API Gateway stack (in `infra/`) that proxies the Spotify API to return the currently playing track.

**Frontend (`src/`):**
- `pages/` — file-based routing; `index.astro` is the homepage, `about.astro` and `projects.astro` are sub-pages
- `layouts/Layout.astro` — base HTML shell shared across pages
- `layouts/PageLayout.astro` — breadcrumb layout (`home / title`) used by all sub-pages; wraps `Layout.astro`
- `components/Sphere.astro` — canvas wrapper that loads `lib/sphere.ts`
- `components/CurrentlyPlaying.tsx` — React component that polls the API every 30s; falls back to `mock/mock_response.json` when `PUBLIC_API_URL`/`PUBLIC_API_KEY` env vars are absent
- `lib/sphere.ts` — Three.js scene (dual sphere: inner solid + outer wireframe, continuous rotation)
- `styles/global.css` — Tailwind import + CSS custom properties for the color palette

**Environment variables (frontend):**
- `PUBLIC_API_URL` — API Gateway endpoint
- `PUBLIC_API_KEY` — API key for the gateway

**Infrastructure (`infra/`):**
- Single CDK stack (`currently-playing-stack.ts`) wires a Lambda handler to API Gateway
- Lambda code lives in `infra/lib/lambda-handler/`
- DynamoDB table used to cache the currently playing track

## Style

- Color palette: background `#00171F`, body text `#BFBFBF`, highlight/links `#247BA0` (`highlight-text` class), borders `border-zinc-700`, secondary text `text-gray-500`
- Font: Roboto Mono Variable throughout
- Tailwind CSS v4 (via `@tailwindcss/vite`); use the `sm:` breakpoint for mobile layouts
- React is used only where interactivity is needed (`client:load` directive); prefer `.astro` components otherwise
- Timeline pattern (used in `about.astro` experience section): flex row with a left column (`flex-col items-center`) containing a circle dot and `flex-1 w-px bg-zinc-700` line, and a right `flex-1` content column

---
> Source: [lukemcdowell/lukemcdowell.github.io](https://github.com/lukemcdowell/lukemcdowell.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
