## tech-stack

> Project tech stack and architecture


# Tech stack

- **Astro 6.x** (SSR, `output: 'server'`), Node.js 24, port 4321
- **Vanilla JS** only — zero client-side frameworks, no React/Vue/Svelte
- **Tailwind CSS 4** — dark/light themes, mobile breakpoint at 768px
- **ESLint 10** + typescript-eslint + eslint-plugin-astro

# Architecture

```
Browser → vanilla JS fetch → /api/* (Astro SSR proxy) → NextDNS API
```

- API token stays server-side only (`.env`), never sent to browser
- Two proxy endpoints: `GET /api/devices`, `GET /api/logs`
- Whole page scrolls naturally (no internal scroll containers)
- Docker: multi-stage build (`Dockerfile`), env via `--env-file .env`, port 4321

---
> Source: [maciejaszex/ndexplorer](https://github.com/maciejaszex/ndexplorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
