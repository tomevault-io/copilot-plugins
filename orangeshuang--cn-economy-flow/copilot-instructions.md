## cn-economy-flow

> This file provides guidance to the AI agent when working with code in this repository.

# AGENTS.md

This file provides guidance to the AI agent when working with code in this repository.

## Project

A single-page React app visualizing China's economy fund flows as an interactive ECharts graph. All domain content (nodes, flows, labels, stats) is in Chinese. No backend — pure static frontend.

## Commands

- `npm run dev` — Vite dev server (start.sh uses port **5175**, not default 5173)
- `npm run build` — `tsc -b && vite build` (TypeScript project-references build, then Vite)
- `npm run lint` — **oxlint** (not ESLint)
- No test framework is configured

## Key conventions

- `verbatimModuleSyntax` is on — use `import type` for type-only imports.
- Data files (`src/data/nodes.ts`, `src/data/flows.ts`, `src/data/report.ts`) are the single source of truth for all economic statistics. Flow amounts are in **亿元** (100M CNY).
- Graph layout is **manually positioned** via relative coordinates in `src/components/graph/layout.ts` (layered top-to-bottom: decision → fiscal/monetary → market intermediaries → financial → real economy). Not auto-layout.
- Edge curveness is tuned per-edge in `EDGE_CURVENESS` to separate parallel/reverse edges — adjust carefully and test visually.
- ECharts is tree-shaken via `echarts/core` — only import needed chart types and components.

---
> Source: [OrangesHuang/cn-economy-flow](https://github.com/OrangesHuang/cn-economy-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
