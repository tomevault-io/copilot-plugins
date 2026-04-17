## personalredirectinspectorwebapp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install       # Install dependencies
npm run dev       # Start dev server (http://localhost:5173/PersonalRedirectInspectorWebApp/)
npm run build     # TypeScript check + production build (dist/)
npm run preview   # Preview production build locally
npm run deploy    # Build and deploy to GitHub Pages
```

There is no test suite in this project. Use `npm run build` to catch TypeScript and build-time errors.

## Architecture

This is a **purely client-side** React + TypeScript + Vite app. All state is stored in `localStorage`; there is no backend on the `main` branch (a full-stack AWS version exists on the `aws-deployment` branch).

**Entry point:** `index.tsx` (at the repo root, not inside `src/`) contains the `App` component, all core state, and the `ReactDOM.createRoot` call. It handles:
- URL auto-inspection on load (using `window.location.href` and `useEffect`)
- `localStorage` read/write for history (`redirectHistory` key) and default monitored URL (`defaultCustomMonitorUrl` key)
- Deduplication: a new entry is only added if it differs from the most recent entry
- Manual URL inspection and default URL management

**`src/` structure:**
- `types.ts` — shared `RedirectData` and `KeyValue` interfaces
- `useCopyToClipboard.ts` — custom hook for clipboard operations
- `components/AppHeader.tsx` — header with auto-inspected URI display, manual inspect input, default URL input, clear history button
- `components/RedirectCard.tsx` — card for a single history entry; uses `DataBlock` and `ParamsGrid`
- `components/DataBlock.tsx` — displays a labeled data field with a copy button
- `components/ParamsGrid.tsx` — renders query params as a key/value grid
- `components/HeaderBanner.tsx` — branding/title banner

**Key URL logic:** On load, if `window.location` has no meaningful query params or hash, and a `defaultCustomMonitorUrl` is set in `localStorage`, that default URL is inspected instead of the page's own URL. This is the core "redirect target" use case.

**Routing/base path:** Vite is configured with `base: '/PersonalRedirectInspectorWebApp/'` for GitHub Pages deployment. The `@` alias resolves to the repo root.

**Styling:** MUI v5 with `sx` prop throughout; light theme only.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BrandonLC2020) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
