## panama-stopover-sveltekit-site

> Cloudflare Pages + static SvelteKit build conventions.


# Cloudflare Pages + Static SvelteKit

## Static build expectations
- Prefer prerendered routes.
- Avoid Node-only APIs at runtime.
- Keep environment usage compatible with Cloudflare Pages build/runtime.

## When adding routes
- Ensure new routes can be prerendered or clearly document why not.
- For dynamic routes, provide a strategy (e.g., known entries list or CMS-driven build-time generation).

## Build behavior
- Keep adapter/static conventions intact.
- Avoid introducing server endpoints unless the project explicitly supports them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/copa-airlines-digital-marketing)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/copa-airlines-digital-marketing)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
