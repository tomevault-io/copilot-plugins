## myoatlas

> Interactive 3D muscle anatomy viewer with strength rating and nerve innervation mapping.

# MyoAtlas

Interactive 3D muscle anatomy viewer with strength rating and nerve innervation mapping.

## Tech Stack

- **Vanilla JS** (ES modules, no framework)
- **Three.js** 0.170 — 3D rendering, raycasting, materials
- **jsPDF** — PDF export
- **Vite** 6 — dev server and bundler
- **Netlify** — deployment target

## Project Structure

```
myoatlas/
  index.html              # Single-page app, right sidebar layout
  package.json            # "myoatlas", deps: three, jspdf, vite
  vite.config.js          # base: '/', manualChunks splits three.js
  netlify.toml            # Build + cache headers for GLBs
  public/
    anatomy.glb           # 467 muscle/tendon meshes (24MB, BodyParts3D + Z-Anatomy)
    skeleton.glb           # Bone meshes (9.4MB)
    mesh_mapping.json      # Mesh metadata (name, FMA ID, source)
    favicon.svg
  src/
    main.js               # App entry: scene, raycasting, UI, controls (~1000 lines)
    bodyBuilder.js         # GLB loading, coordinate transform, material setup
    muscleData.js          # MUSCLE_GROUPS + ANATOMY_DB (~100 muscles, 1200 lines)
    nerveData.js           # NERVE_GROUPS (10 peripheral nerves, pattern matching)
    ratingSystem.js        # 5-level rating with localStorage persistence
    exportService.js       # JSON + PDF export
    styles.css             # Slate-900/teal design system, single sidebar
```

## Key Architecture

- **Material sharing**: One MeshStandardMaterial per muscle group, not cloned per mesh
- **Coordinate transform**: BP3D (mm, Z-up) → Three.js (scene units, Y-up) in bodyBuilder.js
- **Rating keys**: Multi-part muscles share one rating key (e.g. all heads of triceps → "Left Triceps Brachii")
- **Nerve mapping**: Pattern matching in nerveData.js maps mesh rawNames to nerve keys
- **Visible mesh cache**: Raycasting uses cached array, invalidated on visibility changes
- **localStorage**: Ratings persist across sessions under key `myoatlas_ratings`

## Commands

```bash
npm run dev      # Dev server on localhost:4000
npm run build    # Production build to dist/
npm run preview  # Preview production build
```

## Design System ("Midnight Glass")

- Canvas background: `#0F1115`
- Panel background: `rgba(24,27,33,0.85)` with `backdrop-filter: blur(24px) saturate(140%)`
- Accent: `#3B82F6` (blue-500); selected mesh: `#60A5FA`
- Font: Inter 400/500/600/700 (Google Fonts)
- Borders: `rgba(255,255,255,0.1)` 1px rim on all glass surfaces
- Layout: 64px top bar + floating 320px left + 360px right sidebars (12px margin, 16px radius)
- Rating palette (1-5, MRC-style): `#EF4444` · `#F97316` · `#F59E0B` · `#84CC16` · `#10B981`
- Muscle list active state: 3px blue left pill + `rgba(59,130,246,0.1)` tint
- Rating control: segmented (flex row, recessed black/40 bg, active = grade-color fill + glow)
- Export PDF: capsule primary button, accent glow shadow

**Legacy CSS aliases in `:root`:** `--text`, `--text-muted`, `--bg-dark`, `--accent-light` point to the new tokens so existing references keep working.

**Deferred from redesign.md (architectural cost):**
- Per-mesh hover emission boost (would break shared-material perf optimization).
- Selection ghosting (non-selected muscles to opacity 0.3) — conflicts with the `muscle-opacity-slider` which mutates shared materials; needs a base-opacity + multiplier refactor to coexist.

## Data Sources

- **BodyParts3D** — CC BY-SA 2.1 Japan (The Database Center for Life Science)
- **Z-Anatomy** — CC BY-SA 4.0 (Gauthier Kervyn)
- **Nerve mapping** — Based on clinical neuromodell document

## Landmines

- GLB files are large (24MB + 9.4MB). Netlify headers set immutable caching for `*.glb`.
- `boneMaterial` has `depthWrite: false` to avoid z-fighting with transparent bones.
- Muscles can belong to multiple nerves (e.g. flexor digitorum profundus → median + ulnar).
- The `mesh_mapping.json` names use spaces but GLTFLoader converts spaces to underscores in node names — bodyBuilder.js normalizes for lookup.
- Rating materials are shared instances. Opacity changes via sliders mutate them in place.

## Mobile UX (2026-04-24)

Complete mobile redesign with touch-first interface. See `MOBILE-UX.md` for full details.

**Key features:**
- Two-finger pinch zoom (touch event handlers, raycasting at midpoint)
- Bottom sheet rating modal (slides up, 56px touch targets, swipe-to-dismiss)
- FAB menu (quick access to export/reset/clear, staggered animations)
- Full desktop compatibility (scoped via `@media (max-width: 1024px)` and `isMobile()`)

**Architecture:**
- `isMobile()` helper: `window.innerWidth <= 1024`
- Mobile sheet: z-index 200, max-height 80vh (60vh landscape)
- FAB menu: z-index 150, staggered reveal with 50ms delays
- Desktop selection card hidden on mobile: `display: none !important`

**Code locations:**
- Touch zoom: `main.js` lines 51-100, 168-249
- Mobile sheet: `main.js` lines 372-621, `styles.css` lines 1327-1625
- FAB menu: `main.js` lines 1230-1263, `styles.css` lines 1627-1728

---
> Source: [bendeguzkudor/myoatlas](https://github.com/bendeguzkudor/myoatlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
