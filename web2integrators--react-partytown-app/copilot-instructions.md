## react-partytown-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Performance tools monorepo for React applications. Provides packages for off-main-thread analytics (Partytown), real-time Web Vitals monitoring, PageSpeed Insights integration, and Lighthouse CI tooling.

## Commands

```bash
# Install dependencies
pnpm install

# Development (all packages + demo app)
pnpm dev

# Build all packages
pnpm build

# Lint and typecheck
pnpm lint
pnpm typecheck

# Clean all build artifacts
pnpm clean

# Run Lighthouse CLI (after build)
pnpm lighthouse <url> [options]
node packages/lighthouse/dist/cli.js <url> --strategy=mobile --budget=90
```

**Single package development:**
```bash
cd packages/web-vitals && pnpm dev    # Watch mode for one package
cd apps/demo && pnpm dev              # Demo app only
```

## Architecture

**Monorepo Structure:**
- `apps/demo/` - React demo app consuming all packages
- `packages/` - Publishable packages under `@perf-tools/*` namespace

**Package Dependency Graph:**
```
demo
├── @perf-tools/partytown-analytics
├── @perf-tools/web-vitals → @perf-tools/shared
└── @perf-tools/pagespeed-insights → @perf-tools/shared

@perf-tools/lighthouse (standalone Node.js CLI)
```

**Key Packages:**

| Package | Purpose |
|---------|---------|
| `partytown-analytics` | Google Analytics running in Web Worker (off-main-thread) |
| `web-vitals` | Real-time Core Web Vitals via PerformanceObserver (LCP, CLS, INP, TTFB, FCP, TBT) |
| `pagespeed-insights` | PageSpeed Insights API React components |
| `lighthouse` | CLI for performance audits with CI/CD budget enforcement |
| `shared` | Common types and utilities |

**Build Tooling:**
- **Turbo** orchestrates tasks with dependency graph awareness
- **Vite** bundles demo app with Partytown plugin (outputs `dist/~partytown/`)
- **tsup** bundles library packages as dual ESM + CJS with type declarations

## Key Patterns

**Partytown Integration:** GA scripts execute in a Web Worker to avoid blocking the main thread. The demo app supports toggling between Partytown (`?ga=partytown`) and standard (`?ga=standard`) modes via URL parameter.

**Web Vitals Collection:** Uses `PerformanceObserver` API to capture metrics in real-time. The `useWebVitals` hook aggregates LCP, CLS, INP, TTFB, FCP, and custom TBT calculation.

**Lighthouse CI:** The CLI supports performance budgets for CI/CD gating:
```bash
# Fails if performance score < 90
pnpm lighthouse https://example.com --budget=90
```

## Environment Variables

Copy `.env.example` to `.env`:
```
VITE_GA_MEASUREMENT_ID=G-XXXXXXXXXX
```

## TypeScript

Strict mode enabled. Base config in `tsconfig.base.json`, extended by each package. Target ES2020 with `react-jsx` transform.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Web2Integrators)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Web2Integrators)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
