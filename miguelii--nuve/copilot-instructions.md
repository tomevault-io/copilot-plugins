## nuve

> Premium supercar showcase built with Next.js 16, React 19, TypeScript 6, Three.js, and Tailwind CSS 4.

# Nuvē

Premium supercar showcase built with Next.js 16, React 19, TypeScript 6, Three.js, and Tailwind CSS 4.

## Stack

- **Framework:** Next.js 16.2.1 (App Router, Turbopack)
- **Language:** TypeScript 6 (strict mode)
- **3D:** Three.js + @react-three/fiber + @react-three/drei
- **Styling:** Tailwind CSS 4 (PostCSS)
- **Animation:** Motion (Framer Motion), Lenis (smooth scroll)
- **Icons:** Lucide React
- **Env validation:** @t3-oss/env-nextjs + Zod
- **React Compiler:** enabled via babel-plugin-react-compiler

## Commands

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start dev server (Turbopack) |
| `pnpm build` | Production build |
| `pnpm lint` | Run linter via vite-plus (`vp lint`) |
| `pnpm fmt` | Format code via vite-plus (`vp fmt`) |
| `pnpm typecheck` | TypeScript check (`tsc --noEmit`) |
| `pnpm check` | Lint + typecheck |
| `pnpm knip` | Detect unused code/dependencies |

## Tooling

- **Linter/Formatter:** vite-plus (replaces ESLint + Prettier) — config in `vite.config.ts`
- **Unused code detection:** knip — config in `knip.json`
- **Git hooks:** husky — pre-commit runs: `fmt → check → knip`
- **Package manager:** pnpm

## Code Style

- No semicolons
- Single quotes
- Tab width: 4
- Trailing commas: es5
- Print width: 100

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── layout.tsx          # Root layout (metadata, Lenis, providers)
│   ├── page.tsx            # Home page (force-static, 48h revalidation)
│   ├── sitemap.ts          # Dynamic sitemap generation
│   ├── [showroomId]/       # Dynamic car detail pages
│   └── legal-notice/       # Legal notice page
├── components/
│   ├── ui/                 # Reusable UI components
│   ├── header/             # Navigation header
│   ├── footer/             # Footer with socials
│   ├── showcase-section/   # Home page 3D car showcase
│   ├── showroom/           # Individual car detail components
│   └── splash-screen/      # First-visit loading animation
├── env/                    # Type-safe env vars (client.ts, server.ts)
├── lib/                    # Utilities (cn, buildId, CSP)
├── services/               # Business logic (ShowroomService)
├── providers/              # Context providers (HistoryProvider)
├── styles/                 # Global CSS, theme colors/fonts
└── types/                  # TypeScript types (Showroom, GLTFResult)
```

## Key Patterns

- **Path alias:** `@/*` maps to `./src/*`
- **Utility function `cn()`:** combines clsx + tailwind-merge for class names
- **ShowroomService:** static data layer with `getAll()` and `getById()` methods
- **HistoryProvider:** tracks navigation history (used by SplashScreen for first-visit detection)
- **GLSL shaders:** loaded via raw-loader (Turbopack rule in next.config.ts)

## Data

4 car models: Lamborghini SVJ, Porsche GT3 RS, Ferrari SF90, BMW M4.
Static data in `src/services/showroom-data.ts`. GLB models in `public/models/`.

## Changes Log

### Tooling Migration (2026-03-25)
- **Removed:** ESLint (`eslint`, `eslint-config-next`, `@eslint/eslintrc`, `eslint.config.mjs`), Prettier (`.prettierrc`)
- **Removed:** unused dependency `class-variance-authority`
- **Added:** `vite-plus` — unified linting (oxlint) + formatting in `vite.config.ts`
- **Added:** `knip` — unused code/dependency detection with `knip.json`
- **Added:** `husky` — git pre-commit hook (`fmt → check → knip`)
- **Updated scripts:** `lint` → `vp lint`, added `fmt`, `typecheck`, `check`, `knip`, `prepare`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Miguelii) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
