## coophub

> This file provides guidance to AI coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project overview

Coophub aggregates open-source repositories from cooperative enterprises across multiple git forges (GitHub, GitLab, Git.coop, Codeberg, 0xacab). It fetches org/repo data via each forge's API, caches it, and serves a web UI + JSON API at [coophub.io](https://coophub.io).

**Stack:** Elixir/Phoenix (backend) + React 18/TypeScript (frontend), bundled with Webpack 5. Caching via [Cachex](https://github.com/whitfin/cachex). HTTP client is HTTPoison.

## Common commands

All commands run from the repo root. Use `Makefile` targets where available.

```bash
# Setup
make setup          # deps.get + compile + npm install

# Development server (auto-watches JS/CSS via webpack)
mix phx.server                                       # → localhost:4000

# Tests + checks
make test            # format check + mix test + dialyzer
mix test                                             # run tests only
mix test test/coophub/repos_test.exs:LINE            # run single test at line

# Type checking (dialyzer)
make dialyzer        # incremental
make plt             # full rebuild PLT

# Production build
make release         # webpack prod + digest + mix release
make start           # start release
make stop            # stop release
```

**Docker alternative:** `docker compose up -d` (reads optional `GITHUB_OAUTH_TOKEN` from `.env`).

## Architecture

### Data flow

```
cooperatives.yml ──→ CacheWarmer ──→ Backends ──→ Git forge APIs
                          │                 (HTTPoison)
                          ▼
                    Cachex (ETS)
                   repos_cache + uris_cache
                          │
                          ▼
                    Coophub.Repos  ──→ RepoController ──→ JSON API
                   (query/aggregate)       │
                                           ▼
                                    PageController ──→ React SPA
```

### Key modules

**`Coophub.CacheWarmer`** — OTP-based cache warmer. Reads `cooperatives.yml`, groups orgs by source, fetches org+repo+topic+language data in parallel per source via `Task.async`, stores everything in Cachex. Persists to `repos-cache.dump` between restarts. GitHub orgs are rate-limit aware: only fetches as many as the remaining API quota allows.

**`Coophub.Backends`** (behaviour) — Multi-forge abstraction. Each backend module (`Backends.Github`, `Backends.Gitlab`, `Backends.GitCoop`, `Backends.Codeberg`, `Backends.HexAcab`) implements callbacks: `prepare_request_org/1`, `parse_org/1`, `prepare_request_repos/2`, `parse_repo/1`, `prepare_request_topics/2`, `parse_topics/1`, `prepare_request_languages/2`, `parse_languages/1`. `Gitlab` handles self-hosted instances; `GitlabCom` delegates to it with `gitlab.com` base URL.

**`Coophub.Repos`** — Pure query/aggregation layer over the cache. Provides sorting (by popularity or latest activity), search (regex across name/description/topics/languages, AND/OR modes), language aggregation with percentage calculation, and a popularity scoring algorithm (see below).

**`CoophubWeb.RepoController`** — JSON API controller with **URI-level response caching**. Each endpoint uses `maybe_get_response_from_cache/3` which memoizes responses in the `uris_cache` keyed by `path+query_string`. If the cache entry expires, it re-queries `Coophub.Repos` and commits the result.

**`CoophubWeb.Plug.SubdomainMatcher`** — Redirects `<subdomain>.coophub.io` → `coophub.io/orgs/<subdomain>` (if a cooperative with that key exists) or `coophub.io/languages/<subdomain>` (otherwise).

### Popularity scoring

Implemented in `Coophub.Repos.get_repo_popularity/1`. Combines stargazers (×1.5), forks (×1.7), and open issues (×0.3), with a 0.5× penalty for forks. Divides by a time-decay factor (`hours_since_update ^ 1.8`), then adds 80% of that quotient back. Org popularity = sum of its repos' scores.

### Schemas

- `Coophub.Schemas.Organization` — cooperative org, key fields: `key`, `login`, `name`, `repos` (list of Repository), `languages` (list of `%{"lang" => name, "percentage" => float}`), `popularity`, `last_activity`, `yml_data` (raw entry from cooperatives.yml)
- `Coophub.Schemas.Repository` — single repo, key fields: `key` (parent org key), `name`, `topics`, `languages`, `popularity`, `fork`, `stargazers_count`, `forks_count`

### Frontend

React 18 SPA served by a single Phoenix template (`lib/coophub_web/templates/page/index.html.eex`). Routing via `react-router-dom` v5:

| Path | Component | Purpose |
|------|-----------|---------|
| `/` | `pages/index` | Home page with stats, language chart, popular repos |
| `/orgs` | `pages/orgs` | All cooperatives listing |
| `/orgs/:name` | `pages/org` | Single org detail + repos |
| `/topics/:topic` | `pages/topics` | Repos by topic |
| `/search` | `pages/searchResults` | Search results |
| `/languages/:lang` | `pages/languageRepos` | Repos by language |

Key dependencies: `reactstrap` (Bootstrap 4), `chart.js` + `react-chartjs-2` (language charts), `react-icons`, `lodash`. Entry point: `assets/js/app.tsx` → `Root.tsx` → `BrowserRouter`.

## Configuration

- **`cooperatives.yml`** — Data source. Each entry: `key: { source, login, name, url, description, location }`. This is the file contributors edit to add their cooperative.
- **`config/config.exs`** — Base config: `fetch_max_repos: 10` (dev), Cachex names, cache intervals
- **`config/dev.exs`** — Webpack watcher, live reload, port 4000
- **`config/prod.exs`** — `fetch_max_repos: 100`, `cache_interval: 60` min, HTTPS host
- **`config/test.exs`** — Uses `:repos_cache_test` (isolated from dev cache), no server
- **`GITHUB_OAUTH_TOKEN`** — Env var for authenticated GitHub API requests (higher rate limit). Not required but strongly recommended.

## Testing

ExUnit tests live in `test/coophub/`. Test fixtures are in `test/support/fixtures.ex` — a large file with mock API response data for all backends. Tests use `Cachex` with `:repos_cache_test` namespace (isolated from dev). Run with `mix test`.

## CI

Two GitHub Actions workflows:
- **`test.yml`** (on push): deps.get → format check → `mix test` → dialyzer
- **`build.yml`** (on release published): same checks → webpack prod build → `mix release` → tar.gz artifact → dispatches deploy

## Adding a backend

1. Create `lib/coophub/backends/<name>.ex` implementing the `Coophub.Backends` behaviour
2. Add `get_backend_module!/1` clause in `lib/coophub/backends/backends.ex`
3. Add `source` mapping in `cooperatives.yml` entries

## Adding a cooperative

Edit `cooperatives.yml` — add entry under the org key with `source`, `login`, `name`, `url`, `description`, `location`. No code changes needed.

---
> Source: [fiqus/coophub](https://github.com/fiqus/coophub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
