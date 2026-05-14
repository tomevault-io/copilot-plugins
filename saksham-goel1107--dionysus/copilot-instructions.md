## nextjs

> - Default to React Server Components; limit `use client` to components that need browser APIs or interactivity

- Default to React Server Components; limit `use client` to components that need browser APIs or interactivity
- Use route handlers for server endpoints under `src/app/**/route.ts`
- Use `generateMetadata`/`metadata` for head data; avoid legacy `next/head`
- Keep image domains in sync with `images.remotePatterns` in `next.config.js`
- Optimize performance: stream where feasible, defer non-critical JS via dynamic import
- Prefer `cache`, `revalidate`, and `fetch` options instead of ad-hoc caches
- Respect middlewares defined in `src/middleware.ts` for auth and security; do not duplicate logic in routes

---
> Source: [Saksham-Goel1107/Dionysus](https://github.com/Saksham-Goel1107/Dionysus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
