## backstage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Backstage application - an open platform for building developer portals. Backstage is configured for the `darrylbowler72` organization with GitHub integration for catalog discovery and authentication.

## Environment Requirements

- Node.js: 20 or 22
- Package manager: Yarn 4.4.1 (Berry)
- Database (dev): SQLite (better-sqlite3) with in-memory storage
- Database (prod): PostgreSQL

## Common Commands

### Development
```bash
yarn install          # Install dependencies (always run after checkout)
yarn start           # Start both frontend (port 3000) and backend (port 7007) in dev mode
yarn tsc             # Type check all packages
yarn tsc:full        # Full type check without skipLibCheck
```

### Building
```bash
yarn build:backend   # Build backend package only
yarn build:all       # Build all packages
yarn build-image     # Build Docker image (tagged as 'backstage')
```

### Testing
```bash
yarn test            # Run tests for changed packages
yarn test:all        # Run all tests with coverage
yarn test:e2e        # Run Playwright end-to-end tests
```

### Code Quality
```bash
yarn lint            # Lint changed files since origin/master
yarn lint:all        # Lint entire codebase
yarn fix             # Auto-fix linting issues
yarn prettier:check  # Check code formatting
yarn clean           # Clean build artifacts
```

### Scaffolding
```bash
yarn new             # Interactive CLI to create new plugins/packages
```

## Architecture

### Monorepo Structure

This is a Yarn workspaces monorepo with two main package locations:
- `packages/` - Core application packages (app, backend)
- `plugins/` - Custom plugins (currently empty, reserved for internal plugins)

### Backend Architecture (packages/backend)

The backend uses Backstage's new backend system with a modular plugin architecture. All plugins are registered in `packages/backend/src/index.ts` using the `backend.add()` pattern.

**Enabled Backend Plugins:**
- **App Backend**: Serves the frontend application
- **Proxy Backend**: API proxy for frontend requests
- **Scaffolder**: Software templates and GitHub scaffolding actions
- **TechDocs**: Technical documentation with Docker-based generation
- **Auth**: GitHub OAuth and Guest authentication providers
- **Catalog**: Software catalog with GitHub discovery and entity model
- **Permission**: Allow-all policy (customize for production)
- **Search**: PostgreSQL search engine with catalog and techdocs collators
- **Kubernetes**: Kubernetes cluster integration
- **Notifications & Signals**: Real-time notifications system

### Frontend Architecture (packages/app)

React-based SPA using Material-UI with Backstage's plugin system. Includes standard Backstage plugins: catalog, scaffolder, TechDocs, search, user settings, org management, and Kubernetes integration.

### Configuration

Configuration uses a layered approach with environment variable substitution:
- `app-config.yaml` - Base development config
- `app-config.local.yaml` - Local overrides (git-ignored)
- `app-config.production.yaml` - Production config (PostgreSQL, external URLs)

**Required Environment Variables:**
- `GITHUB_TOKEN` - GitHub PAT for repository access
- `GITHUB_CLIENT_ID` / `GITHUB_CLIENT_SECRET` - OAuth app credentials
- Production: `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_USER`, `POSTGRES_PASSWORD`

### GitHub Integration

The catalog is configured to automatically discover repositories in the `darrylbowler72` organization:
- Scans for `catalog-info.yaml` files
- Runs every 30 minutes
- Filters on `master` branch
- Supports Components, Systems, APIs, Resources, and Locations

## Docker Deployment

The multi-stage Dockerfile builds a production image:
1. Creates dependency skeleton
2. Installs dependencies and builds packages
3. Creates minimal runtime image with production dependencies

Build context must be repository root:
```bash
# From packages/backend directory:
docker build ../.. -f Dockerfile --tag backstage

# Or use the npm script:
yarn workspace backend build-image
```

The runtime container expects:
- Config files: `app-config.yaml` and `app-config.production.yaml`
- Examples directory (included for demo/reference)
- Environment variables for secrets and database connection

## Development Workflow

1. **Starting Development**: Run `yarn install` then `yarn start` to launch both frontend and backend
2. **Adding Plugins**: Use `yarn new` to scaffold new plugins, or manually add to `packages/backend/src/index.ts`
3. **Configuration Changes**: Edit `app-config.yaml` for dev or create `app-config.local.yaml` for personal overrides
4. **Testing Changes**: Run `yarn test` for unit tests, `yarn test:e2e` for integration tests
5. **Type Safety**: Always run `yarn tsc` before committing to catch type errors

## Key Technical Details

- **Backend Port**: 7007 (both dev and prod default)
- **Frontend Port**: 3000 (dev only, served by backend in production)
- **Database**: In-memory SQLite for dev, PostgreSQL for production
- **Node Options**: `--no-node-snapshot` required for Scaffolder compatibility (Node 20+)
- **TechDocs**: Uses Docker for local document generation
- **Permission Model**: Currently allow-all (customize before production use)
- **Branch Strategy**: Main branch is `main`, but organization repos may use `master`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darrylbowler72) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
