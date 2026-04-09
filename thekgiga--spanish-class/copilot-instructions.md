## spanish-class

> Auto-generated from all feature plans. Last updated: 2026-02-25

# spanish-class Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-25

## Active Technologies
- TypeScript 5.4+ (ES2020), React 18 (011-premium-education-ui-redesign)
- N/A (UI-only changes) (011-premium-education-ui-redesign)

- TypeScript 5.4+ (Node.js 18+ for backend, ES2020 for frontend)
- Prisma ORM (Database schema and migrations)
- React + Vite (Frontend)
- Express.js (Backend API)

## Project Structure

```text
spanish-class/
├── packages/
│   ├── backend/          - Express API server
│   ├── frontend/         - React frontend
│   └── shared/           - Shared types and utilities
├── scripts/              - All automation scripts (organized by function)
│   ├── build/           - Build and packaging scripts
│   ├── deploy/          - Deployment automation
│   ├── database/        - Database migration and initialization
│   └── README.md        - Comprehensive scripts documentation
├── docs/                - Project documentation
├── specs/               - Feature specifications
└── e2e/                 - End-to-end tests
```

## Scripts Organization

**IMPORTANT:** All automation scripts are located in `/scripts/` directory, organized by function.

### Script Categories

- **`scripts/build/`** - Build and packaging operations
  - `build-deploy-package.sh` - Creates deployment packages for backend/frontend

- **`scripts/deploy/`** - Deployment automation
  - `deploy-dev.sh` - Deploy to development environment
  - `deploy-prod.sh` - Deploy to production environment
  - `deploy.sh` - Generic deployment script
  - `deploy-multi.sh` - Multi-environment deployment

- **`scripts/database/`** - Database operations
  - `init-remote-db-dev.sh` - Initialize/reset development database
  - `migrate-remote-dev.sh` - Apply migrations to development
  - `migrate-remote-prod.sh` - Apply migrations to production

**Documentation:** See `scripts/README.md` for detailed usage, safety levels, and decision trees.

## Development Rules

### Script Management

1. **All new scripts MUST be placed in `/scripts/{category}/`**
   - Choose appropriate category: build, deploy, or database
   - Follow naming convention: `{action}-{target}.sh`
   - Examples: `backup-database.sh`, `deploy-staging.sh`

2. **Script Requirements**
   - Include descriptive header comments
   - Use absolute paths or proper working directory handling
   - Add safety confirmations for destructive operations
   - Use colored output (GREEN=success, RED=danger, YELLOW=warning)
   - Provide clear error messages
   - Exit with non-zero code on failures

3. **Documentation**
   - Document new scripts in `scripts/README.md`
   - Include purpose, usage, prerequisites, safety level
   - Add to decision tree if applicable
   - Update this file (CLAUDE.md) if adding new categories

### Environment Configuration

**System:** Backend uses `ENV` variable to load environment-specific configs from `config/` directory.

**Structure:**
```
config/
├── local/.env    # Local development (ENV=local)
├── dev/.env      # Dev server (ENV=dev)
├── prod/.env     # Production (ENV=prod)
└── templates/    # Templates for generating configs
```

**Local Development Workflow:**
```bash
# Initial setup (once)
cp config/templates/.env.local.template config/local/.env
nano config/local/.env  # Fill in actual values

# Daily development
npm run dev  # Automatically uses ENV=local → config/local/.env
```

**Deployment Workflow:**
```bash
# Initial setup (once per environment)
cp config/templates/.env.dev.template config/dev/.env
nano config/dev/.env  # Fill in actual values

# Deploy (configs copied automatically)
./scripts/build/build-deploy-package.sh
./scripts/deploy/deploy-dev.sh  # Includes config/dev/.env automatically
./scripts/database/migrate-remote-dev.sh  # Only if schema changed
```

**Production:**
```bash
# Setup (once)
cp config/templates/.env.prod.template config/prod/.env
nano config/prod/.env  # Fill in actual values (different secrets!)

# Deploy (test in dev first!)
./scripts/deploy/deploy-prod.sh  # Includes config/prod/.env automatically
./scripts/database/migrate-remote-prod.sh  # EXTREME CAUTION!
```

**Key Points:**
- Configs are **never committed** to git (entire `config/` is git-ignored)
- Deployment scripts **automatically copy** the correct config
- Backend loads from `config/$ENV/.env` based on `ENV` variable
- NPM scripts set `ENV=local` for local development
- Deployment scripts set `ENV=dev` or `ENV=prod` on the server
- See `config/README.md` for detailed configuration documentation

### Database Operations

- **Local development:** Use `npx prisma migrate dev` in `packages/backend/`
- **Remote databases:** Use scripts in `scripts/database/` (run from local machine)
- **Never run Prisma CLI on cPanel** - Memory limits prevent it from working
- **Migrations are pre-generated** during build and included in deployment package

### cPanel Deployment Notes

- Backend deployed to `~/spanish-class-{env}/`
- Frontend deployed to `~/public_html/{env}/`
- `node_modules` must be a symlink (managed by cPanel Node.js Selector)
- Shared packages placed in `_shared_lib/` with post-install symlinks
- Prisma client pre-generated locally (not on server)
- Restart Node.js app in cPanel UI after deployment

## Commands

```bash
# Development
npm run dev          # Start all packages in dev mode
npm test            # Run all tests
npm run lint        # Lint all packages

# Build
npm run build       # Build all packages

# Database (local)
cd packages/backend
npx prisma migrate dev        # Create and apply migration
npx prisma migrate status     # Check migration status
npx prisma studio            # Open Prisma Studio
```

## Code Style

- TypeScript 5.4+ (Node.js 18+ for backend, ES2020 for frontend)
- Follow standard TypeScript conventions
- Use ESLint for code quality
- Use Prettier for formatting (if configured)

## Recent Changes
- 011-premium-education-ui-redesign: Added TypeScript 5.4+ (ES2020), React 18

- 2026-02-25: Organized all scripts into `/scripts/` directory with categories
- 2026-02-25: Created comprehensive scripts documentation in `scripts/README.md`

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thekgiga)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thekgiga)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
