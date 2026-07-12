## service-map

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # start dev server (Vite HMR)
pnpm build        # tsc type-check + Vite production build → dist/
pnpm lint         # ESLint
pnpm preview      # serve the dist/ build locally
```

No test runner is configured. Type-checking is part of `build` via `tsc -b`.

## What this project does

A React + ReactFlow visualization of inter-service dependencies for the DTP team. Services are described in YAML files, loaded at build time via Vite's `import.meta.glob`, validated against a Zod schema, laid out automatically with ELK (Eclipse Layout Kernel), and rendered as an interactive graph.

## Data model and schema

All service data lives in `data/`:
- `data/services/*.yml` — one file per service. Each file is parsed as a single `ServiceSchema` object.
- `data/externals.yml` — a YAML array of external services (third-party systems DTP integrates with).

The Zod schema in [src/schema.ts](src/schema.ts) defines the canonical types. Key constraints:
- `id` must be kebab-case (`/^[a-z0-9-]+$/`) and unique across all services.
- `area` must be one of the `AREAS` const enum.
- `kind` must be one of the `KINDS` const enum.
- `depends_on[].target` must reference a known `id`; unknown targets are collected as runtime errors.
- External services require `kind: external`, `area: External`, and `external: true`.

To add a new service, create `data/services/<kebab-id>.yml`. The loader (`src/loader.ts`) picks it up automatically via glob.

## Data flow

```
data/*.yml  →  loader.ts (loadGraph)  →  App.tsx (state)  →  Canvas.tsx (ELK layout → ReactFlow)
                                       └→  FilterSidebar.tsx
                                       └→  DetailDrawer.tsx
```

`loadGraph()` returns `{ nodes: Service[], edges: GraphEdge[], errors: string[] }`. Validation errors are non-fatal: valid services are still rendered, and errors are shown in a dismissible banner.

## Layout engine

`Canvas.tsx` runs ELK asynchronously on every layout trigger. ELK uses the `layered` algorithm with `RIGHT` direction. Because ELK's default entry uses Node.js web workers, `vite.config.ts` aliases `elkjs` to `elk.bundled.js` (the self-contained browser bundle).

Layout re-runs when:
1. `relayoutKey` increments (manual reset or exiting focus mode).
2. `focusTrigger` changes (entering/updating focus mode).

## Filtering and focus mode

Filtering happens in two layers:
- **Dim mode** (default): all nodes are in the ELK layout; non-matching nodes are dimmed with `opacity: 0.13`.
- **Focus mode**: clicking "Focus view" runs a fresh ELK layout on only the matching nodes. Isolated nodes (no edges to others in the matching set) are excluded from the focus layout.

Hover tracing (BFS in both directions from the hovered node via `getConnectedChain`) overrides filter dimming.

## URL state

Filters, search query, and selected node are serialised to `?q=&areas=&kinds=&statuses=&techs=&node=` via `window.history.replaceState`. The URL is always shareable and reflects the current view.

## Keyboard shortcuts

- `/` — focus the search input
- `Escape` — deselect the current node

## Styling approach

No CSS framework. All styles are inline React style objects. Design tokens (area colours, edge colours, status badge colours, kind icons) are centralised in [src/graph/styles.ts](src/graph/styles.ts). Adding a new area or edge kind requires updating both `schema.ts` (the enum) and `styles.ts` (the visual token).

## Deployment

GitHub Actions builds and deploys to GitHub Pages. The Vite config sets `base: "./"` so the build works at any subdirectory path.

---
> Source: [pkhaninejad/service-map](https://github.com/pkhaninejad/service-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
