## tiny-svg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tiny SVG is a monorepo containing:
1. **Web app** - SVG optimizer and code generator (TanStack Start + React 19)
2. **Figma plugin** - SVG optimization directly in Figma
3. **Shared packages** - Core SVG logic, code generators, UI components

**Tech Stack:**
- Framework: TanStack Start (SSR with file-based routing), React 19
- UI: Tailwind CSS 4, Radix UI, shadcn/ui
- State: Zustand
- i18n: Intlayer (EN, ZH, KO, DE, FR) - web app only
- Build: Vite 7, pnpm workspaces (pnpm@10.14.0)
- Linting: Biome + Ultracite
- Deployment: Vercel (Cloudflare Workers not supported due to MDX `eval()` restrictions)

## Common Commands

```bash
# Development
pnpm dev              # Start all workspace apps
pnpm dev:web          # Start only web app (port 3001)
pnpm dev:figma        # Start Figma plugin in watch mode

# Build
pnpm build            # Build all packages (runs workspace builds in dependency order)
pnpm build:packages   # Build only shared packages (svg, code-generators, utils, ui)
pnpm build:figma      # Build Figma plugin for production
pnpm --filter @tiny-svg/web build   # Build web app only
pnpm --filter @tiny-svg/web serve   # Preview web app production build

# Code Quality
pnpm check            # Run Biome linter/formatter (auto-fix)
pnpm check-types      # TypeScript type checking across all workspaces

# Internationalization (web app only)
pnpm exec intlayer build  # Build i18n dictionaries (run from apps/web)
```

## Workspace Architecture

```
tiny-svg/
├── apps/
│   ├── web/                     # Main web application (TanStack Start)
│   └── figma-plugin/            # Figma plugin (React + Figma Plugin API)
└── packages/
    ├── svg/                     # @tiny-svg/svg - SVGO config, presets, utilities
    ├── code-generators/         # @tiny-svg/code-generators - Framework code generators
    ├── ui/                      # @tiny-svg/ui - Shared React components (diff viewer, etc.)
    └── utils/                   # @tiny-svg/utils - General utilities
```

### Package Dependencies

**Apps depend on packages:**
- `apps/web` → `@tiny-svg/svg`, `@tiny-svg/code-generators`, `@tiny-svg/ui`
- `apps/figma-plugin` → `@tiny-svg/svg`, `@tiny-svg/code-generators`, `@tiny-svg/ui`, `@tiny-svg/utils`

**Important:** After modifying any `packages/*`, run `pnpm build:packages` before building apps.

### Shared Packages

**`@tiny-svg/svg`** (`packages/svg/src/`)
- SVGO default configuration and presets (default-config.ts, presets.ts)
- Compression presets: Default, Aggressive, Minimal, Custom
- SVGO utilities and types
- Built with `tsdown` to CommonJS + ESM

**`@tiny-svg/code-generators`** (`packages/code-generators/src/`)
- Framework code generators: React (JSX/TSX), Vue, Svelte, React Native, Flutter
- Single file: `generators.ts` with all generator functions
- Built with `tsdown` to CommonJS + ESM

**`@tiny-svg/ui`** (`packages/ui/src/`)
- Shared React components (not built, consumed as source via workspace protocol)
- Diff viewer component with syntax highlighting
- Radix UI-based components shared between web and plugin
- Exports paths: `@tiny-svg/ui/components/*`, `@tiny-svg/ui/lib/*`

**`@tiny-svg/utils`** (`packages/utils/src/`)
- General utilities used across projects
- Built with `tsdown` to CommonJS + ESM

## Web App Architecture (`apps/web/`)

```
apps/web/src/
├── components/         # React components (UI, optimize, lazy-loaded)
├── contents/           # i18n definitions (*.content.ts)
├── hooks/              # Custom React hooks
├── lib/                # Utilities and helpers
│   ├── worker-utils/       # Worker client interfaces
│   ├── svg-to-code.ts      # Framework code generators (wraps @tiny-svg/code-generators)
│   ├── svg-transform.ts    # SVG transformations (rotate, flip, resize)
│   ├── svgo-plugins.ts     # SVGO plugin configurations (wraps @tiny-svg/svg)
│   └── file-utils.ts       # File handling utilities
├── routes/             # TanStack Start file-based routing
│   ├── __root.tsx          # Root layout
│   ├── {-$locale}/         # Locale-prefixed routes (e.g., /en, /zh)
│   └── og.tsx              # Open Graph image generation
├── store/              # Zustand stores
│   ├── svg-store.ts        # SVG content, optimization settings, transformations
│   └── ui-store.ts         # UI state (theme, panels, preferences)
└── workers/            # Web Workers for heavy tasks
    ├── svgo.worker.ts      # SVG optimization (SVGO runs here)
    ├── prettier.worker.ts  # Code formatting (Prettier runs here)
    └── code-generator.worker.ts  # Framework code generation
```

## Figma Plugin Architecture (`apps/figma-plugin/`)

```
apps/figma-plugin/
├── assets/             # Plugin assets
│   ├── icon.svg            # Vector icon (40x40)
│   └── icon.png            # Raster icon (128x128)
├── src/
│   ├── plugin.ts       # Plugin sandbox code (Figma API, runs in restricted context)
│   └── ui/             # React UI (runs in browser context)
│       ├── components/     # React components (svg-list, svg-item, preview-drawer, etc.)
│       ├── hooks/          # Custom hooks (use-svg-optimization, use-figma-selection)
│       ├── store/          # Zustand store (plugin-store.ts)
│       └── styles.css      # Figma-styled CSS
├── manifest.json       # Figma plugin manifest
├── vite.config.ts      # Build config (includes icon copying to dist/)
└── dist/               # Built plugin (generated)
    ├── index.html          # UI bundle (inlined via vite-plugin-singlefile)
    ├── plugin.js           # Plugin sandbox code
    ├── icon.svg            # Plugin icon
    └── icon.png            # Plugin icon
```

**Plugin Communication:**
- `plugin.ts` (sandbox) ↔ `ui/` (browser) communicate via `figma.ui.postMessage()` and `window.onmessage`
- Plugin runs SVGO optimization via imported `@tiny-svg/svg` package
- Code generation via imported `@tiny-svg/code-generators` package

## Key Patterns

### Internationalization (Web App Only)
- Define translations in `*.content.ts` files using `t()` function with keys for each locale
- Access translations with `useIntlayer('contentName')` hook
- Supported locales: EN, ZH, KO, DE, FR (configured in `intlayer.config.ts`)
- Routes use `{-$locale}` pattern for locale-based routing (e.g., `/en/about`, `/zh/about`)
- After adding/modifying content files, run `pnpm exec intlayer build` from `apps/web`

### Web Workers (Web App Only)
Heavy operations run in Web Workers to avoid blocking the main thread:
- **Worker files** (`apps/web/src/workers/`): Contain actual worker logic
- **Worker clients** (`apps/web/src/lib/worker-utils/`): Provide type-safe interfaces to communicate with workers
- **Worker manager** (`worker-manager.ts`): Handles worker lifecycle and instance management
- SVGO optimization, Prettier formatting, and code generation all execute in dedicated workers
- **IMPORTANT:** SVGO library is NOT imported in main bundle, only in `svgo.worker.ts` to keep bundle size small

### State Management
**Web App (`apps/web/src/store/`):**
- `svg-store.ts`: SVG content, optimization settings, transformations, SVGO config
- `ui-store.ts`: UI state (theme, panels, preferences)

**Figma Plugin (`apps/figma-plugin/src/ui/store/`):**
- `plugin-store.ts`: Unified store for SVGs, settings, UI state

Uses Zustand for simple, performant state management without boilerplate.

### Code Generation Pattern
Both web app and Figma plugin use `@tiny-svg/code-generators`:
1. Optimize SVG with SVGO
2. Pass optimized SVG to generator function (e.g., `generateReactCode()`)
3. Format with Prettier (web app uses worker, plugin formats directly)
4. Display or copy to clipboard

## Code Style

This project uses **Ultracite** with **Biome** for strict linting. Run `pnpm check` to auto-fix issues.

**Key rules enforced:**
- Use `import type` for type imports
- Use `export type` for type exports
- No TypeScript enums (use `as const` objects instead)
- No `any` type (disabled in biome.json for pragmatic reasons, but avoid when possible)
- No non-null assertions (`!`)
- Use `for...of` instead of `Array.forEach`
- Use arrow functions over function expressions
- Always include `type` attribute on buttons
- Include `title` attribute for accessibility on interactive elements

**Overrides in biome.json:**
- `noExplicitAny`: off (pragmatic exception)
- `noReactForwardRef`: off (React 19 compatibility)
- `noDangerouslySetInnerHtml`: off (needed for SVG rendering)
- Console methods: `warn`, `info`, `error` allowed

## Working with Figma Plugin

**Development workflow:**
1. Build shared packages: `pnpm build:packages`
2. Start plugin dev mode: `pnpm dev:figma` (watches for changes)
3. In Figma Desktop: **Plugins → Development → Import plugin from manifest...** (select `apps/figma-plugin/manifest.json`)
4. After plugin code changes: **Plugins → Development → Reload plugin**

**Important notes:**
- UI changes hot-reload automatically
- Plugin sandbox changes (`plugin.ts`) require manual reload in Figma
- Icons are copied from `assets/` to `dist/` during build via `vite.config.ts`
- Plugin uses `vite-plugin-singlefile` to inline all assets into single HTML

---
> Source: [hehehai/tiny-svg](https://github.com/hehehai/tiny-svg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
