## sayanseliv-github-io

> Personal portfolio/showcase site deployed to GitHub Pages. Stack: Nuxt 4 (SSR → static generation), Vue 3, TypeScript, TresJS (@tresjs/nuxt, @tresjs/cientos, @tresjs/post-processing), GSAP, lucide-vue-next, @nuxtjs/sitemap, @nuxtjs/robots, nuxt-github-pages. SCSS with sass.

# CLAUDE.md — sayanseliv.github.io

## Project Overview

Personal portfolio/showcase site deployed to GitHub Pages. Stack: Nuxt 4 (SSR → static generation), Vue 3, TypeScript, TresJS (@tresjs/nuxt, @tresjs/cientos, @tresjs/post-processing), GSAP, lucide-vue-next, @nuxtjs/sitemap, @nuxtjs/robots, nuxt-github-pages. SCSS with sass.

## Project Structure

```
app/
  components/
    animations/  # Canvas, WebGL, Three.js, GSAP animation demos
      bg-circuit/
      wave-cube/
    cards/       # CardFeature, CardPortfolio, CardTechnology, CardTools
    layout/      # AppHeader, AppModal
    ui/          # Reusable UI primitives: ProgressBar, TagCloud, CircleProgress, AnimatedBlock
      buttons/   # AnimatedButton, AnimatedMatrix, AnimatedWaveButton, MaskButton, etc.
  composables/
    canvas/      # useCanvas, useCanvasAnimations, useCanvasShapes, useCanvasPatterns
    useWebSocket.ts
  pages/
    index.vue
    features/    # Individual feature/demo pages (canvas, 3d, animations, etc.)
    portfolio/
  plugins/
    gsap.client.ts
  types/
    index.ts     # Shared types: Content, Tool, CategoryData
    canvas.ts
    particle-network.d.ts
content/
  data.json      # Technology and tool data (static content)
assets/styles/
  reset.scss
  global.scss
  _mixins.scss   # Auto-imported into all SCSS via vite preprocessor
```

## TypeScript Rules

- Use `interface` for object shapes in `types/index.ts` (existing convention); use `type` for unions, aliases, and utility types.
- Do NOT use `any`. Use proper types, generics, or `unknown` with narrowing.
- Do NOT annotate `ref` when the type can be inferred from the initial value. Only annotate when inference is impossible, e.g. `ref<string | null>(null)`.
- Run `yarn type-check` after any type changes.

## Commands

```bash
yarn dev          # start dev server
yarn build        # SSR build
yarn generate     # static build (used for GitHub Pages deploy)
yarn preview      # preview production build
yarn type-check   # TypeScript check
yarn lint         # ESLint
yarn lintfix      # ESLint with auto-fix
yarn test         # run tests (vitest)
yarn test:run     # run tests once (CI)
yarn test:coverage
```

Package manager is **yarn** (v4). Do not use `npm` commands.

## Key Conventions

- Components registered without path prefix (`pathPrefix: false` in nuxt.config).
- Alias `@` = `app/` (Nuxt 4 default).
- SCSS mixin file `_mixins.scss` is auto-injected into every SCSS block via Vite — no manual `@use` needed in components.
- 3D and WebGL components (TresJS, canvas-heavy) must be wrapped in `<ClientOnly>` and lazy-loaded.
- GSAP is loaded via `app/plugins/gsap.client.ts` — do not import gsap directly in SSR-sensitive paths.
- Icons come from `lucide-vue-next`, not from a custom icon system.
- Static content (technologies, tools) lives in `content/data.json` and is read at build time.

## SEO

- Meta tags are set globally in `nuxt.config.ts` (`app.head`) — no i18n, single language (English).
- Page-level overrides use `useHead()` or `useSeoMeta()`.
- `@nuxtjs/sitemap` is configured with `autoLastmod`, `discoverImages`, `sortEntries`.
- `@nuxtjs/robots` is configured to allow all bots.
- Canonical URL base: `https://sayanseliv.github.io`.
- All `<img>` tags must have meaningful `alt` attributes (enforced by `vuejs-accessibility/alt-text` ESLint rule).
- Hero/above-fold images: `loading="eager"` and `fetchpriority="high"`.

## ESLint / Accessibility Rules (enforced)

- `vue/html-self-closing` — always self-close void, normal, and component tags.
- `vuejs-accessibility/click-events-have-key-events` — all click handlers need keyboard equivalents.
- `vuejs-accessibility/alt-text` — all images must have alt.
- `vuejs-accessibility/label-has-for` — form labels must be associated via nesting or `id`.
- `vuejs-accessibility/form-control-has-label` — all form controls must have a label.

## Testing

- Framework: Vitest + `@nuxt/test-utils` + `@vue/test-utils` + happy-dom.
- Test files live in `tests/componenta/` (note: existing typo in dir name, keep consistent).
- Use `defineVitestConfig` from `@nuxt/test-utils/config` (already set up in `vitest.config.ts`).

## Deployment

- Deployed to GitHub Pages via `nuxt-github-pages` module.
- Nitro preset: `github_pages`.
- Output: `.output/public`.
- Run `yarn generate` to produce the static build.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sayanseliv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
