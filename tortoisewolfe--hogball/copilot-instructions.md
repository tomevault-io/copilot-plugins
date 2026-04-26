## hogball

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Core Development Principles

1. **Proper Solutions Over Quick Fixes** - Implement correctly the first time
2. **Root Cause Analysis** - Fix underlying issues, not symptoms
3. **Stability Over Speed** - This is a production template
4. **Clean Architecture** - Follow established patterns consistently
5. **No Technical Debt** - Never commit TODOs or workarounds

## Docker-First Development (MANDATORY)

**CRITICAL**: This project REQUIRES Docker. Local pnpm/npm commands are NOT supported.

### NEVER Install Packages Locally

**ABSOLUTELY FORBIDDEN** - Never run these commands on the host machine:

```bash
# ❌ CRITICAL NO - NEVER do any of these locally
npm install
npm install --no-save <package>
pnpm install
pnpm add <package>
yarn install
npx <anything>

# ✅ CORRECT - Always use Docker
docker compose exec hogball pnpm install
docker compose exec hogball pnpm add <package>
```

**Why this is critical:**

- Creates local `node_modules` with wrong permissions (Docker-owned)
- Causes conflicts between host and container dependencies
- Breaks the Docker-first architecture
- Creates cleanup nightmares (Docker-owned files can't be deleted by host user)

**If you accidentally installed locally:**

```bash
docker compose down
docker compose run --rm hogball rm -rf node_modules
docker compose up
```

### NEVER Use sudo - Use Docker Instead

When encountering permission errors, **NEVER use `sudo`**. Use Docker:

```bash
# ❌ WRONG - Don't do this
sudo chown -R $USER:$USER .next
sudo rm -rf node_modules

# ✅ CORRECT - Use Docker
docker compose exec hogball rm -rf .next
docker compose exec hogball rm -rf node_modules
docker compose down && docker compose up
```

**Why**: The container runs as your user (UID/GID from .env). Docker commands execute with correct permissions automatically.

**Permission errors? Always try:**

1. `docker compose down && docker compose up` (restarts container, cleans .next)
2. `docker compose exec hogball pnpm run docker:clean`

### Essential Commands

```bash
# Start development
docker compose up

# Development server
docker compose exec hogball pnpm run dev

# Run tests
docker compose exec hogball pnpm test
docker compose exec hogball pnpm run test:suite    # Full suite

# Storybook
docker compose exec hogball pnpm run storybook

# E2E tests
docker compose exec hogball pnpm exec playwright test

# Type checking & linting
docker compose exec hogball pnpm run type-check
docker compose exec hogball pnpm run lint

# Clean start if issues
docker compose exec hogball pnpm run docker:clean
```

### Git Commits from Docker

Git hooks may fail when running locally if the repo was set up inside Docker. Always commit from inside the container:

```bash
# Configure git identity (add to .env)
GIT_AUTHOR_NAME=Your Name
GIT_AUTHOR_EMAIL=your@email.com

# Commit from container (hooks run correctly)
docker compose exec hogball git add -A
docker compose exec hogball git commit -m "Your commit message"

# Push from host (uses your SSH keys)
git push
```

### Supabase Keep-Alive

Supabase Cloud free tier auto-pauses after 7 days. If paused:

```bash
docker compose exec hogball pnpm run prime
```

## Forking This Repository

When forking HogBall (or any ScriptHammer-based project) to create your own project:

### Quick Start

```bash
# 1. Fork and clone
gh repo fork TortoiseWolfe/HogBall --clone
cd YourProject

# 2. CRITICAL: Stop Docker first (service name will change)
docker compose down

# 3. Run rebrand script
./scripts/rebrand.sh YourProject yourusername "Your project description"

# 4. Create .env file
cp .env.example .env
# Edit .env with your UID/GID (run: id -u && id -g)

# 5. Rebuild with new service name
docker compose up --build

# 6. Verify
docker compose exec yourproject pnpm run build
docker compose exec yourproject pnpm test -- --run
```

### Manual Review After Rebrand

The rebrand script handles 95% of changes automatically, but review these manually:

1. **Footer.tsx** - The script changes the template attribution link. Decide whether to:
   - Keep pointing to ScriptHammer (recommended - give credit to the template)
   - Update to your own project

2. **FORKING-FEEDBACK.md** - Add your learnings for future forkers

3. **.env** - Configure your Supabase credentials for full functionality

### Known Quirks

- The rebrand script updates ALL owner/project references, including template attribution
- Docker service name changes require `docker compose down` first
- Contract/integration tests require Supabase credentials (unit tests work without)
- CLAUDE.md docker commands are automatically updated by the script

### Files Modified by Rebrand Script

| Category | Count | Examples |
|----------|-------|----------|
| Config | 10+ | docker-compose.yml, package.json, .env.example |
| Source | 50+ | Components, services, configs |
| Docs | 100+ | All markdown files |
| Tests | 10+ | Fixtures, contract tests |
| Total | ~350 | All ScriptHammer → YourProject |

See `docs/FORKING-FEEDBACK.md` for detailed lessons learned from previous forks.

## Component Structure (MANDATORY)

Components must follow the 5-file pattern or CI/CD will fail:

```
ComponentName/
├── index.tsx                             # Barrel export
├── ComponentName.tsx                     # Main component
├── ComponentName.test.tsx                # Unit tests (REQUIRED)
├── ComponentName.stories.tsx             # Storybook (REQUIRED)
└── ComponentName.accessibility.test.tsx  # A11y tests (REQUIRED)
```

**Always use the generator:**

```bash
docker compose exec hogball pnpm run generate:component
```

See `docs/CREATING_COMPONENTS.md` for details.

## Architecture Overview

- **Next.js 15** with App Router, static export
- **React 19** with TypeScript strict mode
- **Tailwind CSS 4** + DaisyUI (32 themes)
- **Supabase** - Auth, Database, Storage, Realtime
- **PWA** with Service Worker (offline support)
- **Testing**: Vitest (unit), Playwright (E2E), Pa11y (a11y)

## Static Hosting Constraint

This app is deployed to GitHub Pages (static hosting). This means:

- NO server-side API routes (`src/app/api/` won't work in production)
- NO access to non-NEXT*PUBLIC* environment variables in browser
- All server-side logic must be in Supabase (database, Edge Functions, or triggers)

When implementing features that need secrets:

- Use Supabase Vault for secure storage
- Use Edge Functions for server-side logic
- Or design client-side solutions that don't require secrets

**Example**: The welcome message system uses ECDH shared secret symmetry to encrypt
messages "from" admin without needing admin's password at runtime. The admin's
public key is pre-stored in the database, and `ECDH(user_private, admin_public)`
produces the same shared secret as `ECDH(admin_private, user_public)`.

### Key Paths

```
src/
├── app/           # Next.js pages
├── components/    # Atomic design (subatomic/atomic/molecular/organisms/templates)
├── contexts/      # React contexts (AuthContext, etc.)
├── hooks/         # Custom hooks
├── lib/           # Core libraries
├── services/      # Business logic
└── types/         # TypeScript definitions

tests/
├── unit/          # Unit tests
├── integration/   # Integration tests
├── contract/      # Contract tests
├── e2e/           # Playwright E2E tests
└── setup.ts       # Vitest setup

docker/            # Docker configuration
├── Dockerfile     # Main Dockerfile
└── docker-compose.e2e.yml  # E2E testing compose

docs/specs/        # Feature specifications (SpecKit artifacts)
tools/templates/   # Component generator templates
```

## PRP/SpecKit Workflow

For features taking >1 day:

1. Write PRP: `docs/prp-docs/<feature>-prp.md`
2. Create branch: `./scripts/prp-to-feature.sh <feature> <number>`
3. Run SpecKit (full 7-step workflow):
   ```
   /specify → /clarify → /plan → /checklist → /tasks → /analyze → /implement
   ```

### SpecKit Commands

| Command      | Purpose                                              |
| ------------ | ---------------------------------------------------- |
| `/specify`   | Create feature specification from PRP                |
| `/clarify`   | Ask clarifying questions, encode answers into spec   |
| `/plan`      | Generate implementation plan from spec               |
| `/checklist` | Generate custom checklist for the feature            |
| `/tasks`     | Generate dependency-ordered tasks.md                 |
| `/analyze`   | Cross-artifact consistency check (spec, plan, tasks) |
| `/implement` | Execute the implementation plan                      |

See `docs/prp-docs/SPECKIT-PRP-GUIDE.md` for details.

## Common Issues & Solutions

### Permission Errors

**Always use Docker, never sudo:**

```bash
docker compose down && docker compose up
```

### Slow Supabase (10-30 seconds)

Instance paused after inactivity:

```bash
docker compose exec hogball pnpm run prime
```

### Tailwind CSS Not Loading

1. Don't import Leaflet CSS in `globals.css`
2. Import Leaflet CSS only in map components
3. Restart container after CSS changes

### Port 3000 In Use

```bash
docker compose down
lsof -i :3000
kill -9 <PID>
```

## Test Users

**Primary** (required):

- Email: `test@example.com`
- Password: `TestPassword123!`

**Secondary** (optional - for email verification tests):

- Configure in `.env`: `TEST_USER_SECONDARY_EMAIL`, `TEST_USER_SECONDARY_PASSWORD`

## GitHub Actions Secrets

For CI/CD deployment, add these secrets in **Settings → Secrets and variables → Actions → Repository secrets**:

### Required for Deployment

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

### Optional but Recommended

```
NEXT_PUBLIC_GA_MEASUREMENT_ID=G-XXXXXXXXXX
NEXT_PUBLIC_PAGESPEED_API_KEY=your-google-api-key
NEXT_PUBLIC_AUTHOR_NAME=Your Name
NEXT_PUBLIC_AUTHOR_EMAIL=your@email.com
```

See `README.md` for the complete list of available secrets.

## Documentation

| Topic               | Location                               |
| ------------------- | -------------------------------------- |
| Authentication      | `docs/AUTH-SETUP.md`                   |
| Messaging System    | `docs/messaging/QUICKSTART.md`         |
| Payment Integration | `docs/features/payment-integration.md` |
| Security            | `docs/project/SECURITY.md`             |
| Mobile-First Design | `docs/MOBILE-FIRST.md`                 |
| Component Creation  | `docs/CREATING_COMPONENTS.md`          |
| Template Setup      | `docs/TEMPLATE-GUIDE.md`               |
| Testing Guide       | `docs/project/TESTING.md`              |
| Forking Guide       | `docs/FORKING.md`                      |

## Supabase Database Migrations (CRITICAL)

**NEVER create separate migration files.** This project uses a **monolithic migration file**:

```
supabase/migrations/20251006_complete_monolithic_setup.sql
```

### Adding Schema Changes

1. **Edit the monolithic file directly** - Add new tables, columns, indexes to the appropriate section
2. **Use `IF NOT EXISTS`** - All CREATE statements must be idempotent
3. **Add to existing transaction** - New schema goes inside the `BEGIN;`...`COMMIT;` block
4. **Execute via Supabase Management API** - Use `SUPABASE_ACCESS_TOKEN` from `.env`

### Executing Migrations (Claude Code)

**NEVER tell the user to run migrations manually.** Use the Supabase Management API:

```bash
# Check for access token in .env
SUPABASE_ACCESS_TOKEN=<token>
NEXT_PUBLIC_SUPABASE_PROJECT_REF=<project-ref>

# Execute SQL via Management API
curl -X POST "https://api.supabase.com/v1/projects/${PROJECT_REF}/database/query" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT 1"}'
```

**DO NOT:**

- Tell user to copy SQL to dashboard manually
- Install database clients locally (pg, psql, etc.)
- Try direct database connections from Docker (DNS issues)

### Example: Adding a Column

```sql
-- Add to the appropriate table section in the monolithic file
ALTER TABLE user_encryption_keys
ADD COLUMN IF NOT EXISTS encryption_salt TEXT;
```

### Why Monolithic?

- Single source of truth for entire schema
- Can recreate database from scratch with one file
- No migration ordering issues
- Supabase Cloud doesn't support CLI migrations on free tier

**DO NOT:**

- Create files like `032_add_encryption_salt.sql`
- Suggest running SQL snippets piecemeal
- Use Supabase CLI migrations

## Important Notes

- Never create components manually - use the generator
- All PRs must pass component structure validation
- E2E tests are local-only, not in CI pipeline
- Docker-first development is mandatory
- Use `min-h-11 min-w-11` for 44px touch targets (mobile-first)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TortoiseWolfe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
