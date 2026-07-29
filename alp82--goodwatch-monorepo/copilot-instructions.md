## goodwatch-monorepo

> - use Remix loaders for prepopulated data via SSR

# Data Fetching
- use Remix loaders for prepopulated data via SSR
- use TanStack Query for dynamic data fetching
    - do not add custom `staleTime`, prefer the globally set options

# SEO
- use `seededRandom*` methods instead of `Math.random()` to avoid hydration errors
- verify lighthouse score for significant changes
- use proper meta tags and Open Graph tags
- use proper hreflang tags
- use proper canonical tags

# QA & Testing
- The dev server is always running under http://localhost:3003/, don't run it yourself
- Use Chrome Devtools MCP and use the browser to test UI changes and analyze lighthouse scores
- Don't write automated tests

---
> Source: [alp82/goodwatch-monorepo](https://github.com/alp82/goodwatch-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
