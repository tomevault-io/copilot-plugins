## cinderblock

> Cinderblock is a React Native Web design system (untyped JS, ESM) built around

# Cinderblock — notes for coding agents

Cinderblock is a React Native Web design system (untyped JS, ESM) built around
structural spacing components (Stripe > Bounds > Section > Chunk). This is an npm
workspaces monorepo:

- `packages/design-system` — the library itself (`@cinderblock/design-system`)
- `packages/starter-nextjs-*` — Next.js starter templates (Pages Router)
- `packages/create-cinderblock-app` — CLI that scaffolds from the starters

## Hard constraints — do not break these

1. **Pages Router only.** SSR style extraction (`AppRegistry` +
   `react-native-media-query`'s `flush()`) runs in `Document.getInitialProps`.
   Moving to App Router requires reworking the library around
   `useServerInsertedHTML` — a deliberate project, not a config tweak.
2. **Keep `peerDependencies` as ranges** (`^19`, `^15`), never exact pins.
   Exact pins broke consumers that need patched React (Clerk requires
   `react ~19.0.3+`).
3. **Consumers path-alias only stateful singletons** (react, react-dom,
   prop-types, react-native-web, react-native-media-query). Webpack path
   aliases bypass the package `exports` field and break exports-only packages
   (uuid ≥ 11). If you touch the starters' `next.config.js`, preserve this and
   the `nextRuntime === 'edge'` early-return (protects consumers' middleware).
4. **Pages own their structural content.** Shared Page/application shells own
   metadata, global navigation, and overlays, then render children directly.
   Do not put a catch-all Stripe, Bounds, or Section in a shared shell.

## Key docs

- `packages/design-system/docs/nextjs-integration.md` — **read this before
  debugging any consumer integration issue** (transpile/symlink/exports
  landmines, `file:` dep consumption, TS shim, verification checklist).
- `packages/design-system/docs/AGENTS.md` — **start here for component
  usage**: canonical page skeleton (`Stripe > Bounds > Section > Chunk`),
  page/shell ownership, Flex defaults, responsive model, and anti-patterns.
  Verified against source.
- `packages/design-system/docs/recipes.md` — real-world page patterns; live
  versions in the kitchensink starter under `pages/recipes/`.
- `packages/design-system/docs/*.md` — component API references and patterns.
- After editing docs, run `npm run lint-docs` in `packages/design-system` —
  it checks doc code blocks for nonexistent imports and wrong nesting.

## Known state (2026-07)

- Verified working: React 19.1 + Next 15.5 + RNW 0.21.2 + uuid ^14, SSR and
  production builds, no hydration errors (proven in the Outpost app at
  `~/Repos/outpost`, which consumes this repo via a `file:` dep — treat it as the
  live integration testbed).
- Open TODOs: migrate internal touchables to `Pressable` (console deprecation
  warning from RNW), generate TypeScript declarations, App Router support.
- The starters' `package.json`/`next.config.js` may lag the fixes documented
  in `nextjs-integration.md` — sync them when touched (`npm run sync-templates`).

## Verification

There is no test suite. Verify library changes by running a consumer:
`npm run dev:kitchensink` here, or the Outpost app (`cd ~/Repos/outpost && npm run dev`),
then check: dev server boots with middleware, SSR HTML is styled, no hydration
warnings in the browser console, `next build` passes.

---
> Source: [rxb/cinderblock](https://github.com/rxb/cinderblock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
