## 3d-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start Vite dev server
- `npm run build` — production build
- `npm run preview` — preview production build
- `npm run lint` — ESLint (flat config in `eslint.config.js`); fails on any warning (`--max-warnings 0`)

There is no test suite configured.

## Architecture

This is a single-page 3D portfolio built with React + `@react-three/fiber` (Three.js) + Tailwind CSS, deployed on Vercel.

- **Routing**: `src/App.jsx` defines routes via `react-router-dom`. Only `/` (Home) is currently active — `/about` and `/projects` routes are commented out, and `About.jsx`/`Projects.jsx` are unused leftovers from a template.
- **Main scene**: `src/pages/Home.jsx` renders the `<Canvas>` containing the 3D scene: lighting, `Space` (background) and `SpaceStation` (main interactive model). It also drives the typewriter intro text, background audio, and mobile tap-outside-to-close behavior for info panels.
- **3D models**: `src/models/` contains generated Three.js/R3F components (`Space.jsx`, `spaceStation.jsx`, `Astronaut.jsx`). These were exported from Blender via [gltf.pmnd.rs](https://gltf.pmnd.rs/) and then hand-extended — don't regenerate from scratch when making small changes.
- **Remote assets**: `.glb` model files are NOT bundled in the repo; they're fetched at runtime from a Cloudflare R2 bucket (hardcoded URLs like `https://pub-900982d21fee47f8b53d6e8c8ac9a4cf.r2.dev/...`) via `useGLTF`. `vite.config.js` includes `assetsInclude: ['**/*.glb']` for any local glb usage. `*.glb` is also tracked via Git LFS (`.gitattributes`).
- **Interactive markers/stages**: `spaceStation.jsx` defines a `markers` array (id, 3D position, camera `lookAt`/`lookPosition`, label) rendered as `GlowingMarker` components on the space station mesh. Clicking a marker calls `handleMarkerClick`, which sets `currentStage` (lifted up through `Home.jsx`) and animates the camera via `@react-spring/three` (`useSpring` on position/lookAt/lookPos, applied each frame in `useFrame`).
- **Info panels**: `src/components/HomeInfo.jsx` renders content based on `currentStage` (1 = About Me, 2 = Web, 3 = Design, 4 = XR/Game). When adding a new marker/stage, add an entry to `markers` in `spaceStation.jsx` and a corresponding `currentStage` branch in `HomeInfo.jsx`.
- **Constants**: `src/constants/index.js` holds skills/experience/social links/project data for the (currently mostly unused) About/Projects pages — much of it is commented out.
- **Deployment**: `vercel.json` rewrites all paths to `/` for SPA routing.

---
> Source: [baidiwang/3D-Portfolio](https://github.com/baidiwang/3D-Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
