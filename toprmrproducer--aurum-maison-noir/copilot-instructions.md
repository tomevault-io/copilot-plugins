## aurum-maison-noir

> A Cartier-grade, scroll-driven, fully immersive 3D website for a luxury perfume maison

# AURUM — Immersive 3D Perfume House

A Cartier-grade, scroll-driven, fully immersive 3D website for a luxury perfume maison
(**AURUM** — "gold"). Black void, warm liquid-gold light, glass refraction + bloom.
Built for Shreyas (his own house, not Cartier). Reference vibe: Cartier immersive-scroll
experiences (spotlit hero 3D object, "Continue" enter gate, Lenis smooth scroll, GSAP
scroll-driven camera, editorial reveals per chapter).

## Stack (web-native, NO bundler / NO build step)
- **Three.js r0.160** (WebGL) — procedural glass perfume bottles, RoomEnvironment reflections, UnrealBloom.
- **GSAP 3.12 + ScrollTrigger** — scroll-driven camera + reveals.
- **Lenis** — buttered smooth scroll.
- Plain **static** `.html` + `css/` + ES-module `js/`. Loaded via `python3 -m http.server 8788` (or any static host). Libraries via CDN **importmap** in `index.html`.
- Opens/serves with zero install. There is no `node_modules`.

## File structure
```
index.html            # markup (all chapters, loader, nav, footer) + importmap + <script type=module src=js/main.js>
css/styles.css        # full design system (noir + gold), responsive, hover, cursor, reveal pattern
js/config.js          # SOURCE OF TRUTH — export const AURUM = { brand, palette, perfumes[5], nav, seo }
js/scene.js           # export function initScene({canvas,perfumes,palette}) -> { setProgress, focusChapter, resize, dispose, ready, isWebGL }
js/bottles.js         # export function createBottle(spec) -> THREE.Group  (procedural glass/liquid/gold-cap, LatheGeometry profiles)
js/scroll.js          # export function initScroll({onProgress,onChapter}) -> Lenis+ScrollTrigger, native-scroll fallback
js/ui.js              # export function initUI({perfumes}) -> reveals, custom cursor, nav, counters, marquee
js/loader.js          # export function initLoader({assetsReady,onEnter}) -> Continue gate (hard 6s cap)
js/guards.js          # detectWebGL / installImageGuard / installSafetyReveal / noWebGLFallback
js/backgrounds.js     # export function bgDataURI(spec) -> deterministic on-brand SVG gradient data-URI
js/main.js            # entry — wires guards -> scene -> loader -> (on enter) scroll + ui
assets/bottles/       # bottle-1..5.svg branded placeholders (2D fallback + OG); REPLACE with Shreyas's 5 real glass-bottle images
assets/og/ , assets/favicon.svg
```

## Hard rules for this repo (bulletproof — see [[oneshot-prompts-must-be-bulletproof]])
1. **Visible by default.** No `opacity:0` baked into HTML/CSS. Reveal pattern: `<html class="no-js">`; `main.js` swaps to `js`. `.reveal` is fully visible by default; only `html.js .reveal` starts hidden + transitions, `.is-in` reveals. `guards.installSafetyReveal()` force-reveals everything after ~4.5s if IntersectionObserver misfires. Page + hero H1 render with JS disabled.
2. **Never broken images.** `guards.installImageGuard()` swaps any failed `<img>` to a branded `bgDataURI()` placeholder. No empty/broken slots, ever.
3. **No WebGL = still beautiful.** `guards.noWebGLFallback()` adds `html.no-webgl`; CSS shows a noir+gold gradient stage with the 2D bottle SVGs + all copy. Never a blank black screen.
4. **Loader never traps.** Continue button always enables (hard 6s cap even if assets/CDN fail); Enter key works too.
5. **Debug hook:** `window.__aurum = { setProgress, focusChapter, getCameraPos(), getActiveChapter(), bottleCount, isWebGL }` for Playwright verification.

## Status (23 Jun 2026) — v2 LIVE (horizontal rooms): https://aurum-maison-noir.netlify.app
- Vertical hero -> **PINNED HORIZONTAL gallery** (`#rooms`): vertical scroll translates `.rooms-track` sideways, one `.room` per fragrance. Each room = `.room-copy` (left) + 3D bottle (centre — shows through the room's transparent centre over the opaque canvas) + `.room-plate` (right — the VISIBLE atmospheric image). Click any `a[data-room="i"]` -> glides to that room via `lenis.scrollTo` into the pin range. A no-gsap vertical fallback stacks the rooms.
- Engine (`scene.js`): bottles laid along **+X** (`stationPosition`), camera pans sideways with `focusF`; **`setStage(0|1)`** fades bottles for the vertical outro (scroll.js calls 0 on the marquee trigger, 1 across hero+rooms). Canvas stays opaque so bloom + lighting are intact.
- `scroll.js`: GSAP ScrollTrigger pin + horizontal translate + Lenis; emits `onProgress`/`onChapter`/`onStage`; `[data-room]` click handler.
- Images: room `--img` plates use `assets/img/aurum-smoke.png` + `aurum-bg.png`. IMPORTANT: paths MUST be root-absolute `/assets/img/...` because `url()` inside a CSS custom property resolves relative to the **stylesheet** (`/css/`), not the HTML. Logo `assets/img/aurum-logo.png` in the loader (`mix-blend-mode:screen`).
- **0 console errors**; Playwright-verified on prod (scroll -> active chapter + camera track sideways, click-to-room glide, outro clean).

## KNOWN BLOCKER for public launch
- The 5 Meshy GLB bottles have **knockoff brand text baked into their textures** ("ALPHA MALE ARONA", "HALU TE", "EAU DE PARFUM"). It is part of the model and cannot be removed in code. Before any public deploy, replace `assets/models/bottle-01..05.glb` with **text-free / real bottle models** — re-run `gltf-transform optimize <in> assets/models/bottle-0X.glb --compress meshopt --texture-compress webp --texture-size 2048 --simplify false`. The engine loads by `config.js` path, so no code change is needed.

## Phase 2
- Swap in clean/real bottles (above). Optional: layer the atmospheric imagery behind the 3D (needs the canvas set to `alpha:true` + a background image layer — currently the canvas is opaque so section background images would be hidden). Then deploy to Netlify.

## Conventions
- Brand name, palette, and all 5 perfume names/notes/stories/colors live ONLY in `js/config.js`. Change the house name there once and it propagates.
- Keep it static. If anyone reaches for Vite/React/a bundler, that's wrong for this repo.

---
> Source: [toprmrproducer/aurum-maison-noir](https://github.com/toprmrproducer/aurum-maison-noir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
