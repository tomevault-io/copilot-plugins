## leaflet-example

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — Start Vite dev server
- `npm run build` — Type-check with `tsc -b` then build to `docs/` directory
- `npm run lint` — Run ESLint
- `npm run preview` — Preview the production build locally

## Architecture

React 19 + TypeScript + Leaflet app built with Vite. Single-page app with entry point at `src/main.tsx` rendering `<App />`.

`src/App.tsx` uses `react-leaflet` to display an interactive map centered on Tokyo with 5 landmark markers. Marker icons are fixed for Vite by deleting `L.Icon.Default.prototype._getIconUrl` and using `L.Icon.Default.mergeOptions` with explicitly imported images.

The build output goes to `docs/` (not the default `dist/`) for GitHub Pages hosting. The Vite `base` is set to `/leaflet-example/` to match the GitHub Pages path.

---
> Source: [wasabina67/leaflet-example](https://github.com/wasabina67/leaflet-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
