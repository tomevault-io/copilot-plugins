## frankensim-website

> The marketing and documentation site for [FrankenSim](https://github.com/Dicklesworthstone/frankensim), live at [frankensim.org](https://frankensim.org). Next.js 16 (App Router, Turbopack), React 19, TypeScript strict, Tailwind v4, framer-motion, Three.js, Bun.

# AGENTS.md — FrankenSim Website

The marketing and documentation site for [FrankenSim](https://github.com/Dicklesworthstone/frankensim), live at [frankensim.org](https://frankensim.org). Next.js 16 (App Router, Turbopack), React 19, TypeScript strict, Tailwind v4, framer-motion, Three.js, Bun.

## Ground rules (never violate)

1. **Never delete a file without explicit permission** — even a file you created. Move it to a backup or ask; do not delete.
2. **No destructive git/filesystem commands** (`git reset --hard`, `git clean -fd`, `rm -rf`, force-overwrites, `mv` over a real path) unless the user gives the exact command and confirms the irreversible consequences in the same message.
3. **Bun only** — never npm, yarn, or pnpm. `bun install`, `bun dev`, `bun run build`, `bunx tsc --noEmit`, `bun lint`.

## Conventions

- **All content lives in `lib/content.ts`** — the single source of truth (siteConfig, nav, the seven layers, principles, the crate atlas, flagships, comparison, roadmap, glossary, FAQ). Do not scatter content into other files.
- **Numbers are computed, not typed.** `bun scripts/compute-stats.mjs` refreshes `heroStats` (crate count, lines of Rust, tests) and `bun scripts/generate-atlas.mjs` regenerates the crate atlas from the FrankenSim workspace metadata (`FRANKENSIM_DIR`, default `~/projects/frankensim`). Flowing prose uses floored forms ("100+ crates", "160K+ lines") so it does not go stale between runs.
- **Copy is de-slopified:** minimal em-dashes (prefer commas, semicolons, or a recast), and no "not just X, it's Y" / "Here's why" LLM tells. Keep the declarative, technical voice.
- **Design system:** near-black-teal `#04090d`, cyan `#06b6d4`/`#22d3ee` + violet `#a855f7`, glass surfaces, micro-labels (`text-[10px] font-black uppercase tracking-[0.3em]`), Inter + JetBrains Mono. Reuse the shared utilities in `app/globals.css` (`.glass-modern`, `.card`/`.card-hover`, `.field`, `.btn-secondary`).

## The WASM Lab & E2E campaigns

- The kernels live in the `fs-wasm` crate in the FrankenSim workspace (`crates/fs-wasm`), compiled to `public/fs-wasm/` via `wasm-bindgen --target web`. The worker (`public/fs-wasm/worker.js`) hosts them; `lib/use-fs-wasm.ts` is the shared **singleton-worker** hook (one wasm instance for the whole page). One engine (`fs_wasm_bg.wasm`) serves both surfaces below.
- **Two surfaces.** `/lab` (`components/wasm-lab.tsx`) is 30 quick demos in three tiers: `components/wasm/*` (foundations), `components/wasm/frontier/*` (Three.js/WebGL), `components/wasm/deep/*` (upper stack). `/e2e` (`components/e2e-showcase.tsx`) is 10 certified end-to-end campaign demos in `components/wasm/campaign/*`, each given a full editorial section (motivation, composed crates, "only in FrankenSim", live demo, certified result). The campaign editorial copy lives in `lib/content.ts` (`e2eCampaigns`); each entry's `key` maps to its demo component. Keep campaigns split from the Lab — they are heavier and a different genre, and the split keeps each page's simultaneous-demo count down.
- **Demo rules that keep the page smooth:** pause rAF loops off-screen via `useInView` from `lib/use-viz-anim.ts`; never call `setState` inside an animation loop (mutate refs / use `useEasedText`); dispose Three.js geometries/materials/renderer on unmount; cap DPR at 2; respect `prefers-reduced-motion`; keep canvases `w-full max-w-full` inside a `min-w-0` container (no forced min-width, or you reintroduce mobile overflow).
- **Global scroll performance (do not regress):** `useInView` folds a shared gate into its reported visibility, so every demo's animation freezes while the user is actively scrolling and while the tab is hidden, resuming on scroll-idle — a scroll does near-zero per-frame canvas/WebGL work. `LazyViz` defers demo mounts until the scroll settles and mounts the panel nearest the viewport first, keeping the mount/first-paint storm off the scroll critical path. Any new animated demo that gates on `useInView` inherits this for free; do not add rAF loops that bypass it.

## Gotchas

- **framer-motion + SVG:** never put a static `x=`/`y=` attribute on a `motion` element *and* animate `x`/`y` — framer applies the animated value as a `translate` transform on top of the attribute (doubling the position). Position by one method only; animate `cx`/`cy` for circles, or use `motion.g` transforms.
- **Mobile overflow:** grid/flex children need `min-w-0` when they hold wide content (a viz, canvas, code block, or table), or a wide child forces the column past the viewport and the shell clips it.
- **Custom cursor** is gated on `(hover: hover) and (pointer: fine)` — desktop mice only, never touch devices.
- **Route transitions** are instant (no framer page-fade) to avoid a paint-flash-repaint; do not reintroduce an `opacity: 0` page wrapper.

## Verify before shipping

```bash
bunx tsc --noEmit
bun run build
# deploy (prebuilt): vercel build --prod && vercel deploy --prebuilt --prod
```

Check no horizontal overflow at 390px and no console errors on `/` and `/lab` after visual changes.

---
> Source: [Dicklesworthstone/frankensim_website](https://github.com/Dicklesworthstone/frankensim_website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
