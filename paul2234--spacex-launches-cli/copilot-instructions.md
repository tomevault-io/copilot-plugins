## spacex-launches-cli

> A CLI tool for tracking upcoming SpaceX launches, distributed via npm (`npx spacex-launches-cli`). Launch data is pre-fetched from the SpaceDevs API by a data pipeline (GitHub Action) and served as static JSON from GitHub Pages. The CLI fetches this cached data — it never calls the SpaceDevs API directly.

# AGENTS.md

## Project Overview

A CLI tool for tracking upcoming SpaceX launches, distributed via npm (`npx spacex-launches-cli`). Launch data is pre-fetched from the SpaceDevs API by a data pipeline (GitHub Action) and served as static JSON from GitHub Pages. The CLI fetches this cached data — it never calls the SpaceDevs API directly.

## Tech Stack

- **Language**: TypeScript (ES2022, ESM modules)
- **Runtime**: Node.js >= 18
- **CLI framework**: Commander.js
- **Terminal styling**: chalk
- **HTTP**: Native `fetch` (no axios/node-fetch)
- **Build**: tsup (ESM output, single CLI entry point)
- **Testing**: Vitest
- **Linting**: ESLint + Prettier
- **Package manager**: npm
- **Data hosting**: GitHub Pages (static JSON)
- **Data pipeline**: GitHub Actions (cron every 5 minutes)

## Project Structure

```
src/
  cli/
    index.ts             # CLI entry point (Commander program, shebang)
    commands/
      next.ts            # `spacex-launches next` — show next launch + countdown
      list.ts            # `spacex-launches list` — table of upcoming launches
      detail.ts          # `spacex-launches detail <id>` — full launch details
    formatters.ts        # Terminal formatting helpers (tables, countdown, colors)
  client.ts              # Fetches pre-cached JSON from GitHub Pages CDN
  types.ts               # Domain types (Launch, Mission, Pad, Rocket, etc.)
  config.ts              # Configuration constants (CDN URL, timeouts)

pipeline/
  run.ts                 # Pipeline entry point — orchestrates fetch → transform → publish
  fetch.ts               # Calls SpaceDevs LL2 API with pagination
  transform.ts           # Flattens nested API responses → domain types, filters to SpaceX only
  publish.ts             # Writes JSON files to output directory
  api-types.ts           # Raw SpaceDevs LL2 API response types
  config.ts              # Pipeline config (API URL, provider ID, output dir)

tests/
  transform.test.ts      # Transform logic (flattening, filtering, sorting)
  formatters.test.ts     # Countdown, status, date, table formatting
  client.test.ts         # CDN client (mocked fetch)

.github/
  workflows/
    update-launches.yml  # Cron job: fetch from LL2 API → publish to GitHub Pages
```

**Entry point** (built by tsup):
- `src/cli/index.ts` → `dist/cli.js` → `spacex-launches` CLI binary (via `package.json` `bin` field)

**Data flow:**
```
SpaceDevs LL2 API → [GitHub Action pipeline] → GitHub Pages (upcoming.json) → [CLI fetches]
```

## Coding Conventions

- **Module format**: ESM (`"type": "module"` in package.json)
- **Imports**: Use `.js` extensions in relative imports (required for ESM)
- **Naming**: camelCase for variables/functions, PascalCase for types/interfaces, UPPER_SNAKE_CASE for constants
- **Types**: Prefer `interface` over `type` for object shapes. Use `type` for unions/intersections.
- **Error handling**: CLI commands wrap calls in try/catch and print user-friendly messages (no stack traces).
- **No classes**: Prefer plain functions and modules.
- **Null handling**: Use `null` (not `undefined`) for intentionally absent values in domain types.
- **Formatting**: Prettier defaults (2-space indent, single quotes, trailing commas, 100 char line width)

## Development Workflow

```bash
# Install dependencies
npm install

# Build CLI
npm run build          # tsup → dist/cli.js

# Run CLI during development
npx tsx src/cli/index.ts next
npx tsx src/cli/index.ts list
npx tsx src/cli/index.ts detail <launch-id>

# Run data pipeline locally (fetches from LL2 dev API, writes to data/)
npm run pipeline

# Test
npm test               # vitest run
npm run test:watch      # vitest watch mode

# Lint
npm run lint           # eslint + prettier check
npm run lint:fix       # auto-fix
```

**Testing CLI against local data:**
```bash
npm run pipeline                                    # generates data/upcoming.json
npx http-server data -p 8787 --cors -s &            # serve locally
SPACEX_CLI_CDN_URL=http://localhost:8787 npx tsx src/cli/index.ts next
```

**Building for distribution:**
```bash
npm run build          # produces dist/cli.js
npm link               # makes `spacex-launches` command available globally
```

## Data Sources

### Launch Library 2 (LL2) by The Space Devs

- **Production API**: `https://ll.thespacedevs.com/2.3.0/` (used by pipeline in CI)
- **Dev API**: `https://lldev.thespacedevs.com/2.3.0/` (used by pipeline locally, higher rate limits)
- **Docs**: https://thespacedevs.com/llapi
- **Rate limits**: 15 requests/hour (prod free tier), 300 requests/hour (dev)

**Important**: Do NOT use `api.spacexdata.com` — that API is defunct and no longer returns data.

### SpaceX provider ID

SpaceX's `launch_service_provider` ID in LL2 is **121**. The pipeline uses `launch_service_provider__id=121` as a query filter AND post-filters results by provider ID (the API sometimes includes tangentially related launches from other providers).

### Data pipeline

- A GitHub Action runs every 5 minutes
- Fetches all upcoming launches from LL2 with pagination
- Filters to SpaceX-only launches (provider ID 121)
- Transforms deeply nested API responses into flat domain types
- Writes `upcoming.json` to GitHub Pages via the `gh-pages` branch
- Pipeline uses the production LL2 API in CI; dev API locally

### Static JSON structure (GitHub Pages)

```
https://<user>.github.io/spacex-cli-data/
  upcoming.json          # { launches: Launch[], updatedAt: string }
```

### Environment variables

| Variable | Used by | Default | Purpose |
|----------|---------|---------|---------|
| `SPACEX_CLI_CDN_URL` | CLI | `https://paul2234.github.io/spacex-launches-cli-data` | Base URL for cached JSON |
| `LL2_API_URL` | Pipeline | `https://lldev.thespacedevs.com/2.3.0` | SpaceDevs API base URL |
| `OUTPUT_DIR` | Pipeline | `data` | Directory to write JSON output |

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Data source | Launch Library 2 (The Space Devs) | Only actively maintained public API for SpaceX launch data. The former spacexdata.com API is dead. |
| Static JSON cache | GitHub Action → GitHub Pages | CLI users never hit the rate-limited API. Near-zero hosting cost. Users need no API keys or credentials. |
| Post-filter by provider | Filter in transform step after API fetch | LL2's `launch_service_provider__id` filter sometimes leaks non-SpaceX launches. Double-filtering ensures accuracy. |
| API response flattening | Pipeline transforms raw types → domain types | LL2 responses are deeply nested (5+ levels). CLI receives clean, flat data. |
| Single entry point | CLI only (no MCP server for now) | MCP server deferred to a future phase. CLI is the primary interface. |
| Dev API as default locally | `lldev.thespacedevs.com` for local pipeline runs | 20x higher rate limit (300/hr vs 15/hr). Same data as production. |
| ESM only | No CommonJS support | Modern Node.js standard. All dependencies are ESM-compatible. |
| Minimal dependencies | Only `commander` + `chalk` as runtime deps | Published package is ~3KB compressed. No bloat for CLI-only users. |
| Package name | `spacex-launches-cli` on npm, `spacex-launches` bin command | `spacex-cli` was taken on npm. Bin command is shorter for daily use. |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paul2234)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/paul2234)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
