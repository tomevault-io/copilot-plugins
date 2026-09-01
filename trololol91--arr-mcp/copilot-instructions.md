## arr-mcp

> MCP server for managing Sonarr, Radarr, qBittorrent, and Seerr via Claude.ai on mobile.

# arr-mcp

MCP server for managing Sonarr, Radarr, qBittorrent, and Seerr via Claude.ai on mobile.

## Project layout

```
src/
  index.ts              Entry point — validates env, starts HTTP server
  server.ts             MCP server (McpServer), tool registry, UI tool registration
  http-transport.ts     Express server, OAuth 2.1 (mcpAuthRouter), /mcp sessions
  oauth-provider.ts     OAuthServerProvider — in-memory client/code stores, login form
  services/
    sonarr.ts           sonarrGet / sonarrPost / sonarrPut / sonarrDelete
    radarr.ts           radarrGet / radarrPost / radarrPut / radarrDelete
    qbittorrent.ts      qbtGet / qbtPost
    seerr.ts            serrGet / serrPost
    anilist.ts          anilistQuery (GraphQL) / currentSeason helper
    tmdb.ts             tmdbGet / resolveGenreIds / resolveGenreNames / resolveWatchProviderIds
  tools/
    types.ts            ToolModule interface
    sonarr.ts           23 Sonarr tools
    radarr.ts           21 Radarr tools
    qbittorrent.ts      4 qBittorrent tools
    seerr.ts            17 Seerr tools + applyBlocklistFilter helper
    anilist.ts          6 AniList tools + fetchAnilistUI helper
    tmdb.ts             3 TMDB tools (discover_page, discover_movie, discover_tv) + fetchTmdbDiscoverPage/applyCommonFilters helpers
ui/
  sonarr-releases/      Release browser iframe for Sonarr interactive search
    index.html          Vite entry
    main.ts             App class, release table, Grab logic
  radarr-releases/      Release browser iframe for Radarr interactive search
    index.html
    main.ts
  tmdb-discover/        Discovery card grid for TMDB (trending/popular/upcoming/airing/top rated)
    index.html
    main.ts             Poster cards, Request button, Load more pagination
  anilist/              Anime browser card grid (trending/popular/seasonal/search)
    index.html
    main.ts             Cover art cards, score badges, Request via Seerr button
scripts/
  build-ui.ts           Vite build script — bundles all 4 UI apps to single-file HTML
.github/
  workflows/
    ci.yml              Typecheck + build on push/PR to main
    release.yml         Build + push Docker image to GHCR on version tags (v*.*.*)
```

## Tool counts

| Service | Regular tools | UI app tools |
|---|---|---|
| Sonarr | 23 | 1 (interactive search) |
| Radarr | 21 | 1 (interactive search) |
| qBittorrent | 4 | — |
| Seerr | 17 | — |
| AniList | 6 | 4 (trending / popular / seasonal / search) |
| TMDB | 3 | 12 (trending / trending today / trending movies / trending movies today / trending TV / trending TV today / popular movies / popular TV / upcoming / airing TV / top rated movies / top rated TV) |
| **Total** | **74** | **18** → **92 served** |

Regular tools are registered via `server.registerTool()` and appear in `ALL_TOOLS`. UI app tools are registered via `registerAppTool()` from `@modelcontextprotocol/ext-apps/server` and render HTML iframes in Claude.ai chat. `TOOL_COUNT` in `server.ts` must equal `ALL_TOOLS.length + (number of registerAppTool calls)`.

## Adding a new tool

1. Add an entry to the relevant `src/tools/*.ts` array following the `ToolModule` interface in `src/tools/types.ts`.
2. No registration needed — `ALL_TOOLS` in `src/server.ts` is iterated automatically via `server.registerTool()`.
3. For a UI app tool, use `registerAppTool()` in `server.ts` and update `TOOL_COUNT`.
4. Run `npm run typecheck` to verify, then rebuild and redeploy.

## Build & dev

```bash
npm install          # install deps
npm run typecheck    # TypeScript check (no emit)
npm run build:ui     # bundle all 4 UI apps to dist/ui/ (Vite + vite-plugin-singlefile)
npm run build        # build:ui + tsc (produces dist/)
npm run dev          # run with tsx (requires .env file — needs dist/ui/ pre-built)
npm run lint         # eslint --fix
```

## Deployment

Source lives here (`~/Projects/arr-mcp`). Deployment is in `~/Services/arr/docker-compose.yml`.

```bash
cd ~/Services/arr
docker compose build arr-mcp
docker compose up -d arr-mcp
docker exec arr-mcp wget -qO- http://localhost:3000/health
# → {"status":"ok","tools":92}
```

Alternatively pull from GHCR (pushed on version tags):
```bash
docker pull ghcr.io/trololol91/arr-mcp:latest
```

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `MCP_AUTH_TOKEN` | — | Required. Bearer token for MCP auth |
| `ISSUER_URL` | `http://localhost:3000` | OAuth issuer URL — must be HTTPS in production |
| `SONARR_URL` | `http://sonarr:8989` | Sonarr base URL |
| `SONARR_API_KEY` | — | Sonarr API key |
| `RADARR_URL` | `http://radarr:7878` | Radarr base URL |
| `RADARR_API_KEY` | — | Radarr API key |
| `QBT_URL` | `http://gluetun:9081` | qBittorrent WebUI URL (via gluetun network namespace) |
| `SEERR_URL` | `http://seerr:5055` | Seerr/Overseerr base URL |
| `SEERR_API_KEY` | — | Seerr API key |
| `TMDB_API_KEY` | — | TMDB API key — powers all `tmdb_*` discover tools and the discovery UI |
| `MCP_PORT` | `3000` | HTTP port |

## Network

All services communicate on `arr-net` (172.28.0.0/16). qBittorrent shares gluetun's network namespace — always reach it via `http://gluetun:9081`, never `http://qbittorrent:9081`.

## OAuth / session flow

Claude.ai authenticates via OAuth 2.1 authorization code + PKCE:
1. Claude.ai registers at `POST /register`
2. Browser redirects to `GET /authorize` — login form appears
3. User enters `MCP_AUTH_TOKEN` and submits to `POST /oauth/submit`
4. Server redirects back with auth code → Claude.ai exchanges for token
5. Token is `MCP_AUTH_TOKEN` — used as Bearer on all `/mcp` calls

Session ID is **deterministic**: `sha256(token).slice(0,36)`. Same token always produces the same session ID, so Claude.ai reconnects to the same session after a container restart without re-authentication.

## Key implementation notes

**Seerr blocklist filtering** — Seerr's discover API does not apply `blocklistLanguage` / `blocklistRegion` server-side; the web UI filters client-side. `seerr_trending`, `seerr_popular_*`, `seerr_upcoming_movies`, and `seerr_discover_page` call `applyBlocklistFilter()` which fetches `/settings/main` and `/blocklist` in parallel and filters results by language, country, and individual tmdbId before returning. The TMDB discovery tools (`tmdb_*`) are a separate, TMDB-native discover path and are not currently run through this filter.

**Release search filtering** — Sonarr and Radarr return blocklisted releases with `rejected: true`. All four release search paths filter these out before returning.

**UI dark mode** — `prefers-color-scheme` is applied immediately on iframe load as a fallback; `app.onhostcontextchanged` overrides it when Claude.ai reports a theme. This is needed because `onhostcontextchanged` only fires on theme *changes*, not on initial load.

**AniList cover images** — CSP for `https://s4.anilist.co` is declared on the content item in `registerAppResource` (not the resource listing config). Same pattern applies for TMDB images (`https://image.tmdb.org`) in the TMDB discovery UI.

**TMDB discover filters** — `applyCommonFilters()`/`applyYearFilters()` in `tools/tmdb.ts` are shared across `tmdb_discover_movie`, `tmdb_discover_tv`, and the UI's `tmdb_discover_page` pagination helper — add a new filter there once, not per call site. Filters only take effect on `/discover`-backed types (`popular_movies`, `popular_tv`, `upcoming`, `airing_tv`); `trending*` and `top_rated_*` hit TMDB endpoints that don't accept discover-style query params, so those types stay in `noFilterDiscoverTypes` in `server.ts` and the UI filter bar is hidden for them.

**Sonarr/Radarr tags** — Tags are required when adding a series or movie. IDs: 1 = debrid, 2 = local. Use `sonarr_get_tags` / `radarr_get_tags` to list current values.

---
> Source: [trololol91/arr-mcp](https://github.com/trololol91/arr-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
