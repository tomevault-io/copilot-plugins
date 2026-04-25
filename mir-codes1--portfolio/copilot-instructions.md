## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start Vite dev server with HMR
npm run build     # TypeScript check + production build (tsc && vite build)
npm run preview   # Preview the production build locally
```

No linting or test commands are configured.

## Architecture

Interactive 3D portfolio built with React 18 + Three.js (via `@react-three/fiber`). The scene is a **2×1×1 grid of cube nodes** where each face represents a project. Clicking a face triggers a camera transition and opens a dark-glass overlay card.

### State Flow

All global state lives in **`src/store/usePortfolioStore.ts`** (Zustand):
- `selectedFace` — the currently expanded face `{ faceProject, worldPos, worldNormal }` or `null`
- `hoveredFaceId` — face ID under cursor
- `isIdle` / `isCameraReturning` — drive idle rotation and camera transitions

```
CubeNode onClick → setSelectedFace() → CameraController lerps camera toward face
                                     → ExpandedCard renders at fixed screen position
Close / Escape  → setSelectedFace(null) → camera returns to default [3.6, 3.2, 4.5]
```

### Key Components

| File | Role |
|---|---|
| `src/App.tsx` | `<Canvas>` setup, camera defaults (position `[3.6,3.2,4.5]`, FOV 45°), assembles scene |
| `src/components/AssemblyGroup.tsx` | 2×1×1 grid container; handles idle Y-rotation + X-oscillation; writes face color to `mirColors.ts` mutable target each frame |
| `src/components/CubeNode.tsx` | Single node with custom GLSL shader, Lucide glyph textures, hover spring |
| `src/components/CameraController.tsx` | `useFrame` lerp to focused face; offset 0.85 units so node sits left of viewport |
| `src/components/SceneCard.tsx` | Thin wrapper that reads `selectedFace` from store and renders `ExpandedCard` |
| `src/components/ExpandedCard.tsx` | Fixed-position dark-glass card (project detail UI) |
| `src/components/WelcomeTitle.tsx` | Intro text with sequenced CSS keyframe animations; "Mir" gradient driven by RAF loop reading `mirColors.mirGradientTarget` |
| `src/components/CategoryLegend.tsx` | Color-coded legend overlay for the 6 face categories |
| `src/components/PolkaDotBackground.tsx` | SVG polka-dot pattern behind the canvas |
| `src/components/SceneLighting.tsx` | Three.js lights (ambient + directional) |
| `src/components/ui/SpotlightCursor.tsx` | Canvas-based radial spotlight that follows the mouse (zIndex 9999) |
| `src/data/projects.ts` | All face/project data; grid is 2 nodes × 6 faces = 12 entries |

### Data Model

Projects live in `src/data/projects.ts` as a flat array of `FaceProject`:
```typescript
{ id, nodeIndex, faceDir, category, label, description, techTags, url }
// faceDir: '+x' | '-x' | '+y' | '-y' | '+z' | '-z'
// category: 'ml' | 'tools' | 'math' | 'physics' | 'apps' | 'games'
```

### Rendering Notes

- **Shaders:** `CubeNode` uses custom GLSL (not `MeshStandardMaterial`). Per-face color gradients driven by uniforms. Satin texture from `public/textures/satin_diff.png`.
- **Glyph icons:** `src/utils/glyphTextures.ts` converts Lucide SVG strings → `THREE.CanvasTexture` (256×256, cached).
- **Animations:** `@react-spring/three` for hover scale/emissive; `useIdleTimer` (2 s inactivity) enables auto-rotation.
- **Responsive scale:** `src/hooks/useResponsiveScale.ts` debounces viewport changes; desktop ≤ 1.1×, mobile ~80% of shortest dimension.
- **Cross-component color sync:** `src/utils/mirColors.ts` holds `mirGradientTarget` — a mutable shared tuple (no React state). `AssemblyGroup` writes to it each frame based on visible faces; `WelcomeTitle`'s RAF loop reads it to animate the gradient on "Mir".

### Build Config

`vite.config.ts` splits chunks into `vendor-react`, `vendor-three`, `vendor-r3f`, `vendor-post`, `vendor-spring`, `vendor-zustand`. Base path is `./`. Deployed via Vercel (SPA rewrite in `vercel.json`).

TypeScript strict mode is on; path alias `@/` → `src/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mir-codes1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
