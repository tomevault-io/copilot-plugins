## astro

> Building server-side Astro pages

### Guidelines for Astro

- Use content collections with type safety for blog posts, documentation, etc.
- Leverage Server Endpoints for API routes
- Use POST, GET  - uppercase format for endpoint handlers
- Use `export const prerender = false` for API routes
- Use zod for input validation in API routes
- Implement or reuse middleware for request/response modification
- Use image optimization with the Astro Image integration
- Implement hybrid rendering with server-side rendering where needed
- Use Astro.cookies for server-side cookie management
- Leverage import.meta.env for environment variables
- Always check if you're asked to create public or private pages (if public, update `src/middleware/index.ts`) to allow non-auth browsing

---
> Source: [przeprogramowani/ai-rules-builder](https://github.com/przeprogramowani/ai-rules-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
