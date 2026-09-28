## pipeline-gis-studio

> **Generated:** 2026-08-28

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-08-28

## OVERVIEW

Pipeline GIS Studio is a React 19 + TypeScript + Vite single-page app for interactive pipeline network visualization. It renders vector pipelines on MapLibre GL, runs Turf.js spatial analysis, simulates SCADA telemetry, and provides emergency valve-isolation modeling. Packaged for PanCap deployment.

## STRUCTURE

```
./
├── src/
│   ├── components/   # React UI components (see src/components/AGENTS.md)
│   ├── data/         # Mock pipeline networks, basemap styles, medium metadata
│   ├── utils/        # Turf.js spatial helpers
│   ├── App.tsx       # Top-level state container
│   ├── types.ts      # Shared domain types
│   ├── main.tsx      # Vite entry point
│   └── index.css     # Tailwind v4 imports + MapLibre popup overrides
├── package.json
├── tsconfig.json
├── vite.config.ts
└── metadata.json     # AI Studio manifest
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add a new pipeline medium or theme color | `src/data/mockPipelineNetworks.ts` | Update `MEDIUM_META` and `INITIAL_NETWORKS` |
| Change basemap providers or styles | `src/data/mockPipelineNetworks.ts` | `BASEMAP_STYLES` contains full MapLibre style JSON |
| Add spatial operations | `src/utils/turfUtils.ts` | Geodesic length, split, chunk, elevation, isolation |
| Tweak map rendering / layers | `src/components/PipelineMap.tsx` | All MapLibre sources, layers, and interactions |
| Add inspector panels or charts | `src/components/SegmentInspector.tsx` | Uses Recharts for SCADA curves |
| Add top-bar tools or filters | `src/components/Navbar.tsx` | Network chips, search, basemap dropdown |
| Modify risk scoring logic | `src/components/AiDiagnosticsModal.tsx` | `calculateRiskScore` heuristic |
| Run the app | `npm run dev` | Vite on port 3000, host 0.0.0.0 |

## CODE MAP

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `App` | function | `src/App.tsx` | Global state: segments, nodes, selection, modals |
| `PipelineMap` | component | `src/components/PipelineMap.tsx` | MapLibre canvas, layer lifecycle, draw/split interactions |
| `Navbar` | component | `src/components/Navbar.tsx` | Toolbar, network filters, search, basemap switcher |
| `SegmentInspector` | component | `src/components/SegmentInspector.tsx` | Right-side inspector with specs, SCADA, Turf split |
| `SimulationModal` | component | `src/components/SimulationModal.tsx` | Emergency rupture & valve isolation simulator |
| `AiDiagnosticsModal` | component | `src/components/AiDiagnosticsModal.tsx` | Rule-based AI copilot + risk score |
| `DrawPipelineModal` | component | `src/components/DrawPipelineModal.tsx` | Point-and-click new pipeline creator |
| `ElevationProfile` | component | `src/components/ElevationProfile.tsx` | Cross-section chart drawer |
| `NetworkStats` | component | `src/components/NetworkStats.tsx` | Floating KPI banner |
| `INITIAL_NETWORKS` | const | `src/data/mockPipelineNetworks.ts` | All seeded pipeline networks |
| `MEDIUM_META` | const | `src/data/mockPipelineNetworks.ts` | Colors, units, labels per medium |
| `BASEMAP_STYLES` | const | `src/data/mockPipelineNetworks.ts` | MapLibre style definitions |
| `simulateValveIsolation` | function | `src/utils/turfUtils.ts` | Emergency isolation valve logic |
| `generateElevationProfile` | function | `src/utils/turfUtils.ts` | Synthetic elevation/pressure samples |

## CONVENTIONS

- **React style:** Class components are not used; all components are function components typed with `React.FC<Props>`.
- **Path alias:** `@/*` resolves to the repository root (`./`), not `src/`. Imports look like `import X from '@/src/components/X'` or `from '@/src/types'`.
- **Tailwind v4:** Imported via `@import "tailwindcss";` in `index.css`. No separate `tailwind.config`.
- **Styling:** Tailwind utility classes exclusively; no CSS Modules or styled-components.
- **Types:** Shared domain types live in `src/types.ts`; component props are co-located in each component file.
- **Mock data:** All network geometry and telemetry are static mocks; there is no backend API.
- **MapLibre patterns:** Sources and layers are added lazily in `useCallback` updaters; data is refreshed with `setData`.

## ANTI-PATTERNS (THIS PROJECT)

- **Do not modify the HMR block in `vite.config.ts`.** The `DISABLE_HMR` / `watch` logic is required by AI Studio to prevent flicker during agent edits.
- **Do not rely on real geospatial elevation.** `generateElevationProfile` is synthetic sine-wave terrain, not DEM data.
- **Do not add backend routes in `express`.** `express` is a dependency but unused; the app is client-only.
- **Avoid adding real API keys to tracked files.** `GEMINI_API_KEY` must go in `.env.local` (already git-ignored pattern).

## UNIQUE STYLES

- **Bilingual UI:** Toolbar and draw-mode labels use English (the app was previously bilingual with Chinese labels such as "天然气 (Gas)" and "绘制管线").
- **Inline HTML popups:** MapLibre popups are built from template strings with Tailwind classes, not JSX.
- **SCADA live simulation:** Telemetry is generated deterministically on load and then perturbed every 2 s with random noise.
- **Heavy `any` in MapLibre events:** Event payloads and feature properties are frequently cast to `any`.

## COMMANDS

```bash
# Install dependencies
npm install

# Run dev server (port 3000, host 0.0.0.0)
npm run dev

# Production build
npm run build

# Type-check only
npm run lint

# Clean dist artifacts
npm run clean
```

## NOTES

- Requires `GEMINI_API_KEY` in `.env.local` if using server-side Gemini features (AI Studio runtime).
- `DISABLE_HMR=true` disables file watching in Vite; AI Studio sets this automatically.
- No test suite, no ESLint, no Prettier config present.
- Project has no `index.html` in the root listing because Vite's default template is managed by the build harness.

---
> Source: [Coding4Me/pipeline-gis-studio](https://github.com/Coding4Me/pipeline-gis-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
