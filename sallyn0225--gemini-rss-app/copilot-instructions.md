## gemini-rss-app

> **Generated:** Sun Jan 11 2026

# PROJECT KNOWLEDGE BASE

**Generated:** Sun Jan 11 2026
**Updated:** Mon Mar 03 2026 (Cloudflare Pages migration)

## OVERVIEW
Gemini RSS Translator: A React 19 + Vite SPA for RSS aggregation, AI translation, and media proxying. Supports two deployment targets:
- **Vercel Functions + Neon PostgreSQL** (original)
- **Cloudflare Pages Functions + D1 SQLite / Neon PG** (new)

## STRUCTURE
```
.
├── api/             # Vercel Serverless Functions (Backend, legacy)
├── functions/       # Cloudflare Pages Functions (Backend, new)
│   ├── _middleware.ts   # CORS + security headers + error boundary
│   └── api/             # Thin wrappers → server/handlers/
├── server/          # Platform-agnostic shared backend logic (new)
│   ├── db/              # Dual DB: D1 (SQLite) + Neon PG, Repository pattern
│   ├── handlers/        # Core handlers returning Web API Response
│   ├── env.ts           # Cloudflare bindings type definition
│   ├── security.ts      # SSRF/security (no Node.js deps)
│   ├── http.ts          # secureFetch + streamWithSizeLimit
│   └── rate-limit.ts    # KV + InMemory rate limiting
├── components/      # React UI Components (Frontend)
├── db/              # Drizzle ORM Schema & Migrations (Vercel/Neon path)
├── lib/             # Shared Security & HTTP Utilities (Vercel path)
├── services/        # Business Logic (Gemini AI, RSS Processing)
├── scripts/         # Maintenance & Migration Scripts
├── App.tsx          # Main Application Orchestrator
├── index.tsx        # Frontend Entry Point
└── types.ts         # Shared TypeScript Definitions
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| UI / Styling | `components/`, `App.tsx` | Tailwind CSS + Framer Motion |
| API (Vercel) | `api/` | Vercel Node.js functions |
| API (Cloudflare) | `functions/api/` → `server/handlers/` | CF Pages Functions → shared handlers |
| DB Schema (PG) | `db/schema.ts`, `server/db/schema.pg.ts` | Managed via Drizzle Kit |
| DB Schema (D1) | `server/db/schema.d1.ts` | SQLite equivalent |
| DB Client Factory | `server/db/client.ts` | D1 preferred → Neon PG fallback |
| Repository | `server/db/repository.ts` | Unified DB operations interface |
| Security (Vercel) | `lib/security.ts` | DNS-rebinding & private IP protection |
| Security (CF) | `server/security.ts` | No Node.js deps, CF fetch blocks private IPs |
| AI Prompting | `services/geminiService.ts` | Translation & Analysis logic |
| Rate Limiting | `server/rate-limit.ts` | KV-backed + InMemory fallback |
| CF Config | `wrangler.toml` | D1, KV bindings (IDs are placeholders, injected by CI) |
| CF Routing | `public/_routes.json` | Only `/api/*` → Functions |
| CI/CD | `.github/workflows/deploy-cloudflare.yml` | Auto-deploy to CF Pages on push to `main` |

## CONVENTIONS
- **Dual Backend**: Vercel code in `/api` + `/lib`, CF code in `/functions` + `/server`. Don't mix.
- **Flat Source**: Frontend source files (`App.tsx`, `index.tsx`) live in the root, not `/src`.
- **Media Architecture**: Use `MediaUrl` interface; backend provides dual (original/proxied) URLs.
- **Localization**: UI text is primarily **Simplified Chinese**.
- **State Management**: Local state/Context + `IndexedDB` (via `idb-keyval`) for large data; no Redux/Zustand.
- **Repository Pattern**: Handlers never touch Drizzle tables directly — always go through `Repository`.

## ANTI-PATTERNS (THIS PROJECT)
- **DO NOT** use `any` in TypeScript unless required for Drizzle schema compatibility (existing pattern in original code).
- **DO NOT** log or hardcode `ADMIN_SECRET` or API keys.
- **DO NOT** commit `.env` or secrets.
- **NEVER** use raw `<img>` tags without `selectMediaUrl` utility.
- **DO NOT** refactor backend security without unit-testing SSRF safeguards.
- **DO NOT** import Node.js modules (`dns`, `net`, `http`) in `server/` code — it runs on CF Workers.

## UNIQUE STYLES
- **Animations**: Standardized Material-like ease (`easeStandard [0.4, 0, 0.2, 1]`) via `components/animations.tsx`.
- **Styling**: Flat UI aesthetic using custom `accent` and `flat` palettes.

## COMMANDS
```bash
npm install                    # Setup
npm run dev                    # Local Vite dev server
npm run build                  # Frontend build verification
vercel dev                     # Full Vercel environment (legacy)
npm run preview:cf             # Build + local CF Pages dev server
npm run deploy:cf              # Build + manual deploy to CF Pages (needs real IDs in wrangler.toml)
npm run db:generate:d1         # Generate D1 migrations
npm run db:migrate:d1:local    # Apply D1 migrations locally
npx drizzle-kit push           # Sync PG schema to Neon
```

## NOTES
- **SSRF Shield (Vercel)**: `api/feed.ts` and `api/media/proxy.ts` use `resolveAndValidateHost` to block internal network access.
- **SSRF Shield (CF)**: CF Workers fetch automatically blocks requests to private/internal IPs. `server/security.ts` provides URL validation without Node.js dns/net.
- **DB Transactions**: Neither `neon-http` nor D1 (via Drizzle) supports transactions; multi-write operations are sequential.
- **Dual Database**: `server/db/client.ts` auto-detects D1 binding → Neon PG fallback. Repository handles schema differences internally.
- **Rate Limiting**: KV-backed (`expirationTtl` for auto-cleanup) with InMemory fallback when KV is unavailable.
- **Cache Policy**: RSS feeds use `s-maxage` (30m) edge caching.
- **Performance**:
  - High-frequency UI states (e.g., `pullDistance`) are localized to sub-components.
  - Large data (e.g., `read_articles`) is stored in IndexedDB to avoid main-thread blocking.
  - Core dependencies are bundled locally to optimize LCP.

---
> Source: [Sallyn0225/gemini-rss-app](https://github.com/Sallyn0225/gemini-rss-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
