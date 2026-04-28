## ocentra-games

> description: Endpoints ONLY in endpoint-domain — no arbitrary paths anywhere

---
description: Endpoints ONLY in endpoint-domain — no arbitrary paths anywhere
globs: src/**/*.ts,src/**/*.tsx,vite/**/*.ts,packages/card-games/**/*.ts,packages/card-games/**/*.tsx,scripts/**/*.ts,infra/cloudflare/src/**/*.ts
alwaysApply: true
---

# Endpoints Single Source of Truth (MANDATORY)

**Endpoints (local or remote) are ONLY allowed in `@ocentra/endpoint-domain`.**

The rest of the app, Vite plugins, card-games, infra, and scripts **MUST** consume from endpoint-domain. No arbitrary endpoint strings anywhere.

---

## Rule

**FORBIDDEN:**
- `fetch('/api/...')` with raw string
- `` `${workerUrl}/api/matches/${id}` `` or any path concatenation
- `'/api/v1/health'`, `'/local/api/logs'`, etc. as literals
- Defining paths in app, vite, card-games, or infra

**REQUIRED:**
- Import from `@ocentra/endpoint-domain/constants/local` or `@ocentra/endpoint-domain/constants/cloudflare`
- Use `LocalApiEndpoint.*`, `ApiEndpoint.*` for paths
- Use `buildApiUrl(path, { baseUrl })` when combining with worker origin
- Use `QueryParam.*` for query param names

---

## Where Endpoints Live

| Constant | Scope | Use When |
|----------|-------|----------|
| `LocalApiEndpoint` | Vite dev, local NDJSON logs | Main app fetching from Vite dev server |
| `ApiEndpoint` | Cloudflare Worker | Main app fetching from worker, Worker routes |
| `buildApiUrl(path, options)` | Full URL | Combining path + origin (worker, baseUrl) |

---

## Exceptions (Rare)

- **External APIs** (OpenRouter, Ollama): May live in ai-domain or provider config. Not app routes.
- **Vite proxy config** (`vite.config.ts`): Proxy target path must match ApiPathPrefix; path string in proxy is config, not request.
- **Test helpers**: Prefer endpoint-domain constants; hardcoded paths in e2e assertions allowed only when testing the constant itself.

---

## Violations Block Merge

If you add or change an endpoint, add it to endpoint-domain first. Then wire consumers. Never the reverse.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ocentra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
