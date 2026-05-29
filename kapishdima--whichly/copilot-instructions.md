## whichly

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Product

**Whichly** is an open-source React library that mounts a floating "variant picker" on a page. Developers wrap blocks in `<Block>` / `<Variant>`, share a staging link, and clients toggle between variants live on the real page. State is encoded in the URL (`?vp=block:variant,...`) so links are shareable.

There is no backend, no dashboard, no auth — just a single React npm package and a landing page that dog-foods it.

## Repository layout

```
whichly/
├── apps/
│   └── web/                  # Next.js landing page (dog-foods @whichly/react)
├── packages/
│   └── react/                # @whichly/react — the library
│       ├── src/
│       │   ├── index.ts                # exports
│       │   ├── provider.tsx            # WhichlyProvider: state, URL sync, shadow DOM portal
│       │   ├── block.tsx               # <Block name="...">
│       │   ├── variant.tsx             # <Variant name="..."> with display:contents wrapper
│       │   ├── url.ts                  # ?vp= parse/serialize
│       │   ├── ordering.ts             # sort blocks by DOM document position
│       │   ├── lib/utils.ts            # cn() helper
│       │   └── picker/
│       │       ├── index.tsx           # picker UI (steppers + minimize)
│       │       ├── picker.css          # Tailwind v4 entry; injected into shadow root
│       │       ├── shadow-context.tsx  # exposes shadow root to descendants (Radix Portal)
│       │       └── ui/                 # vendored shadcn (Button, DropdownMenu)
│       └── vite.config.ts              # lib mode (ESM + CJS + dts)
├── package.json              # pnpm workspaces
├── pnpm-workspace.yaml
├── biome.json
└── tsconfig.base.json
```

## Architecture notes

- **One package, one tree.** No CDN script, no separate runtime. The picker mounts via `createPortal` into a shadow root appended to `document.body`.
- **Render-all + CSS hide.** Every `<Variant>` is in the React tree. The inactive ones get `display: none`; the active gets `display: contents` so they don't add layout boxes inside flex/grid parents.
- **Shadow DOM isolation.** Picker styles (Tailwind v4 + vendored shadcn) compile to a CSS string via `?inline` and inject into the shadow root at mount. Host page and picker can't bleed styles either way.
- **Tailwind v4 + Shadow DOM.** v4 emits `:root, :host` for theme vars and rewrites `html` in preflight to `:host`, so no manual postprocessing.
- **Radix Portals.** Vendored shadcn `DropdownMenu` reads the shadow root from `ShadowRootContext` and passes it to `DropdownMenuPrimitive.Portal` via the `container` prop, so dropdowns stay inside the shadow root.
- **RSC compatibility.** `WhichlyProvider`, `Block`, `Variant` are `"use client"`. Server Components can import `Block`/`Variant` directly. The namespaced `Whichly` object only works from client components (Next.js client references don't expose properties on Server Components).

## Stack

- **Node** 24 LTS, **pnpm** 9.x
- **TypeScript** 5.x, `strict: true`, `moduleResolution: "Bundler"`
- **Biome** for lint/format
- **packages/react**: Vite library mode + `@tailwindcss/vite` + `vite-plugin-dts`. Peer deps: React 18 || 19, react-dom 18 || 19.
- **apps/web**: Next.js 15 App Router + Tailwind v4 (PostCSS).

---
> Source: [kapishdima/whichly](https://github.com/kapishdima/whichly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
