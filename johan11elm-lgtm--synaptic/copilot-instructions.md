## synaptic

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version (16.2.4) has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Project: portfolio

Personal portfolio for a senior front-end engineer focused on WebGL, shaders, and interactive interfaces. Single-page scroll site: Hero → Services → Work → About → Contact. Hero hosts a refractive lens orb over a noise-driven background sphere; Services renders a smaller orb per service tile; the rest is layout + Tailwind.

## Stack

- **Next.js 16.2.4** (App Router) on **React 19.2.4** — both new enough that training-data patterns may be wrong
- **TypeScript** strict, path alias `@/*` → repo root
- **Tailwind CSS v4** via `@tailwindcss/postcss` — config lives in `app/globals.css` under `@theme inline`, NOT in a `tailwind.config.ts`
- **Three.js 0.184** with `@types/three`
- **ESLint 9** flat config (`eslint.config.mjs`)

## Commands

```bash
npm run dev    # next dev (port 3000)
npm run build  # next build
npm run start  # next start
npm run lint   # eslint
```

No turbopack override, no custom port, no test runner yet.

## Structure

```
app/                 # Next App Router (layout.tsx, page.tsx, globals.css)
components/
  sections/          # Hero, Services, Work, About, Contact — PascalCase .tsx
  webgl/
    LensScene.ts          # Hero: bg sphere (BackSide) + lens orb via CubeCamera
    ServiceOrbScene.ts    # Services: per-tile smaller orb, same Fresnel/dispersion technique
lib/webgl/
  clock.ts           # rAF loop helper, no THREE dep — used by both scenes
  quad.ts            # createFullscreenQuad() — kept as scaffolding for screen-space effects, not currently wired
shaders/
  lens-bg/           # Hero background: simplex noise 3D, 2 base colors + accent
  lens-orb/          # Hero orb: Fresnel + chromatic dispersion (3 refraction vectors)
  service-bg/        # Services tile background variant
  service-orb/       # Services tile orb variant
public/work/         # Project assets (Réviz screens, composites)
analysis/            # Untracked — design references / source-site studies (e.g. monopo-vn)
```

## Conventions worth knowing

- **Shaders live in `.ts` files, not `.glsl`.** GLSL is a tagged template string prefixed with `/* glsl */` for editor highlighting, exported as default. Do NOT introduce `.glsl` files or a webpack loader unless asked. (`.glsl` files under `analysis/` are reference-only, not part of the build.)
- **WebGL is class-based** (see `LensScene.ts`, `ServiceOrbScene.ts`); React components are functional.
- **Any WebGL host component must be `"use client"`** — Three.js touches `window` / `document`.
- **Refractive-orb pattern**: `PerspectiveCamera` + `SphereGeometry` × `BackSide` for the background, plus a second sphere on a separate `Layer` rendered via `CubeCamera` into a `WebGLCubeRenderTarget` for the lens orb. Shader does Fresnel + 3-vector chromatic dispersion. Both scenes share this pipeline.
- **Pause hors-écran**: scenes are gated by `IntersectionObserver` so the rAF loop and cubemap render don't burn cycles when scrolled out of view. Mirror this for any new WebGL scene.
- **`prefers-reduced-motion`**: WebGL hosts must check it and skip mounting the scene entirely (not just freeze a frame).
- **Resize**: components own a `ResizeObserver` and feed `uResolution` (or equivalent) to the shader.
- **Theme tokens** are CSS custom properties (`--color-background`, `--color-foreground`, …) declared in `app/globals.css` under `@theme inline`. Tailwind v4 reads them directly.
- **Fonts** come from `next/font/google` — Geist Sans + Fraunces (with `opsz`, `SOFT` axes and italic) — loaded in `app/layout.tsx`.

## Don't

- Don't downgrade or pin Next/React unless asked.
- Don't add a `tailwind.config.ts` — Tailwind v4 is CSS-first.
- Don't move shaders into `.glsl` files.
- Don't add SSR fallbacks or error boundaries around WebGL "just in case" — host components are already gated client-side.

---
> Source: [johan11elm-lgtm/synaptic](https://github.com/johan11elm-lgtm/synaptic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
