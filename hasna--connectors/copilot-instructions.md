## connectors

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`@hasna/connectors` is a monorepo of 62 TypeScript API connectors with four entry points:

- **CLI** (`src/cli/index.tsx`) — Commander + Ink interactive terminal UI
- **Library** (`src/index.ts`) — Programmatic API for registry, install, search
- **MCP Server** (`src/mcp/index.ts`) — Model Context Protocol server for AI agents
- **Dashboard** (`src/server/serve.ts` + `dashboard/`) — Bun HTTP server serving a Vite/React/shadcn UI

## Commands

```bash
bun install              # Install all deps (dashboard deps auto-install via postinstall)
bun run dev              # Run CLI from source
bun run build            # Build everything: dashboard, CLI, MCP, serve, .d.ts declarations
bun run build:dashboard  # Build dashboard only
bun run typecheck        # tsc --noEmit
bun test                 # Run all 216 tests (6 test files)
bun test src/server      # Run a single test file
npm publish              # Runs prepublishOnly (test + build), then publishes
```

## Architecture

**Build pipeline:** `bun run build` chains: dashboard Vite build → bun build for 4 targets (CLI, MCP, serve, library) → `tsc --emitDeclarationOnly` for `.d.ts` files. CLI externals ink/react/chalk/conf (loaded at runtime).

**Connector registry** (`src/lib/registry.ts`): Static array of 62 `ConnectorMeta` entries with `CATEGORIES` as `const` union type. Versions loaded lazily from each connector's `package.json`.

**Installer** (`src/lib/installer.ts`): Copies connector directories to `.connectors/` in user's project. Auto-generates `.connectors/index.ts` with namespace re-exports (`export * as stripe from './connect-stripe/src/index.js'`). Resolves source connectors from multiple paths (works from both `src/lib/` and `bin/`).

**Auth system** (`src/server/auth.ts`): Reads connector CLAUDE.md files to detect auth type (oauth/bearer/apikey). Manages profiles at `~/.connectors/connect-{name}/`. Two profile file patterns: `profiles/default.json` (flat) and `profiles/default/config.json` + `tokens.json` (directory). OAuth is Google-only currently (hardcoded endpoints + scopes map). CSRF state tokens stored in-memory.

**Dashboard server** (`src/server/serve.ts`): Bun.serve() with route matching via regex. Serves Vite-built SPA from `dashboard/dist/` with fallback to `index.html`. API routes return JSON, static files served by MIME type. Dashboard React app at `dashboard/src/app.tsx` uses @tanstack/react-table.

**Interactive CLI** (`src/cli/components/`): Ink (React for terminals) components. `App.tsx` manages view state machine: main → browse/search → connectors → installing → done. `ConnectorSelect` and `SearchView` share similar table rendering with scroll virtualization.

**MCP Server** (`src/mcp/index.ts`): Registers 8 tools using @modelcontextprotocol/sdk. Wraps the same registry/installer functions the CLI uses. Runs on stdio transport.

## Testing

Tests use Bun's built-in test runner (`import { describe, test, expect } from "bun:test"`). Key patterns:

- **CLI tests** (`cli.test.ts`): Spawn `bun run ./src/cli/index.tsx` as subprocess, parse stdout
- **MCP tests** (`mcp.test.ts`): Spawn MCP server process, send JSON-RPC via stdin/stdout
- **Component tests** (`components.test.tsx`): Use `ink-testing-library` to render and inspect frames
- **Server tests** (`server.test.ts`): Start real Bun.serve() on random port, test with fetch(). Auth tests write to real `~/.connectors/` with unique `zzztest{pid}` prefixed names, cleaned up in afterEach
- **Installer tests** (`installer.test.ts`): Use temp directories (`.test-cli-tmp/`)

Note: Bun's `os.homedir()` caches at startup and ignores runtime `HOME` changes. Tests that need isolated home directories use unique connector names instead.

## Adding a Connector

1. Verify NOT in the blacklist below
2. Create `connectors/connect-{name}/` with: `src/`, `package.json` (`@hasna` scope, `Apache-2.0` license), `CLAUDE.md`, `.npmrc`
3. Add entry to `CONNECTORS` array in `src/lib/registry.ts`
4. Connector names must match `/^[a-z0-9-]+$/` (validated in installer and server)

## Connector Blacklist

Do NOT add these connectors:

**Romanian store/baby scrapers:** connect-aboutyou, connect-altex, connect-answear, connect-carrefour, connect-catena, connect-cel, connect-dedeman, connect-drmax, connect-elefant, connect-emag, connect-evomag, connect-farmaciatei, connect-fashiondays, connect-flanco, connect-footshop, connect-glovo, connect-helpnet, connect-ipb, connect-kaufland, connect-leroymerlin, connect-lidl, connect-mediagalaxy, connect-megaimage, connect-mytheresa, connect-notino, connect-olx, connect-pcgarage, connect-sensiblu, connect-tazz, connect-vivre, connect-babyjohn, connect-jacadi, connect-jumbo, connect-mashashop, connect-minikidi, connect-minikids, connect-nichiduta, connect-noriel, connect-smyk

**Travel scrapers:** connect-googleflights, connect-kayak, connect-kiwi, connect-ryanair, connect-skyscanner, connect-wizzair

**Other:** connect-clickbank, connect-escrow, connect-farfetch, connect-browseruse

## Dashboard API Routes

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/connectors` | GET | All connectors with auth status |
| `/api/connectors/:name` | GET | Single connector details |
| `/api/connectors/:name/key` | POST | Save API key (`{ key, field? }`) |
| `/api/connectors/:name/refresh` | POST | Refresh OAuth token |
| `/oauth/:name/start` | GET | Start OAuth flow (redirects) |
| `/oauth/:name/callback` | GET | OAuth callback handler |

## Auth & Data Storage

Connectors store configuration in `~/.connectors/connect-{name}/`:
- `current_profile` — Active profile name
- `credentials.json` — OAuth client credentials (shared across profiles)
- `profiles/` — Profile JSON files with API keys and tokens

## Publishing

```bash
# 1. Bump version in package.json AND src/cli/index.tsx (.version())
# 2. Update CHANGELOG.md
# 3. Publish (runs tests + build automatically via prepublishOnly)
npm publish
# 4. Update global install
bun install -g @hasna/connectors@<version>
```

The npm package includes `bin/`, `dist/`, `dashboard/dist/`, and `connectors/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hasna)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hasna)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
