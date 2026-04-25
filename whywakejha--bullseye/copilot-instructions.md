## bullseye

> **Mantra: "It needs to look and feel Pixar."**

# Bullseye — Paper Ball Toss Game

**Mantra: "It needs to look and feel Pixar."**

## Stack
- React + Vite
- Three.js via React Three Fiber (`@react-three/fiber`)
- Drei helpers (`@react-three/drei`)
- Rapier physics (`@react-three/rapier`)
- Post-processing (`@react-three/postprocessing`)

## Commands
- `npm run dev` — start dev server
- `npm run build` — production build to `dist/`
- `npm run preview` — preview production build
- `npm run lint` — ESLint

## Golden Rule: Never Build From Scratch
- **ALWAYS repurpose** open-source code, models, and assets before writing anything custom
- Use free GLTF/GLB models from Sketchfab, Poly Pizza, Kenney, Quaternius, etc. for 3D assets
- Reference existing open-source R3F/Three.js games for physics configs and game mechanics
- Pull proven shader code, lighting setups, and post-processing configs from community examples
- Primitive box/cylinder geometry is a last resort, not a starting point
- If a problem has been solved in open source, use that solution

## Conventions
- All 3D components live in `src/components/`
- HTML overlay UI lives in `src/ui/`
- Game state hooks in `src/hooks/`
- 3D models go in `public/models/` as GLTF/GLB files
- Keep components focused: one concern per file
- Pixar-quality lighting and shading is non-negotiable

## Containerization
- `.nvmrc` pins the Node version
- `Dockerfile` + `nginx.conf` for production container
- `package-lock.json` is committed for reproducible installs
- All deps tracked in `package.json` — no global installs assumed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whywakejha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
