## journalvibecoded

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Journal App** - A full-stack personal journaling application using the B-MAD methodology. The app features a guided 4-section entry flow (Professional, Personal, Learning, Gratitude) with AI-powered coaching and pattern recognition. Built with React 19, Node.js/Express, PostgreSQL, and Anthropic Claude API.

**B-MAD Artifacts**: All planning, architecture, and design documents are in `_bmad-output/` directory.

## Repository Structure

```
.
├── client/              # React 19 frontend (Vite + Tailwind)
├── server/              # Node.js backend (Express + Better Auth + Drizzle ORM)
├── scripts/             # Infrastructure validation (Deployment scripts)
├── _bmad-output/        # B-MAD artifacts (PRD, Architecture, UX Design, Epics)
├── docker-compose.yml   # Local development environment
└── package.json         # Root workspace configuration
```

## Common Commands

### Root Level
```bash
npm install                    # Install all dependencies (client + server + root)
npm run test:all              # Run all tests (infrastructure + server)
npm run test:server           # Run server tests only
npm run test:infra            # Run infrastructure validation script
npm run lint:client           # Lint client code
```

### Client (`client/`)
```bash
npm run dev                   # Start Vite dev server (http://localhost:5173)
npm run build                 # Build for production
npm run preview              # Preview production build locally
npm run lint                 # Run ESLint
```

### Server (`server/`)
```bash
npm run dev                  # Start Express dev server with tsx watch
npm run build                # Compile TypeScript to dist/
npm start                    # Run compiled server
npm test                     # Run Vitest test suite
npm run db:migrate           # Run Drizzle migrations
npm run db:rls               # Apply Row-Level Security policies to database
npm run db:generate          # Generate migration files from schema changes
npm run seed:user            # Create test user in database
npm run test:rls             # Test RLS policies
```

### Docker
```bash
docker-compose up -d --build     # Start full stack (server + client + postgres)
docker-compose down              # Stop all services
docker-compose logs -f           # View logs from all services
```

## Setup Instructions

1. **Install dependencies**: `npm install` (installs client, server, and root deps)

2. **Configure environment variables**: Create `.env` in root directory:
   ```
   DATABASE_URL=postgresql://app_user:password@localhost:5432/journal
   DATABASE_ADMIN_URL=postgresql://postgres:password@localhost:5432/journal
   BETTER_AUTH_SECRET=your-secret-key
   ANTHROPIC_API_KEY=your-api-key
   ```

3. **Start Docker environment**: `docker-compose up -d --build`
   - PostgreSQL runs on localhost:5432
   - Server runs on localhost:3000
   - Client dev server runs on localhost:5173

4. **Apply database migrations and RLS**:
   ```bash
   npm run --prefix server db:migrate
   npm run --prefix server db:rls
   ```

5. **Seed test data** (optional): `npm run --prefix server seed:user`

6. **Access the app**: Visit http://localhost:5173

## Architecture

### Application Flow
1. **Frontend** - React router in `client/src/App.jsx` manages: LoginView → HomeView ↔ EntryCreationView
2. **Backend** - Express server with Better Auth and RLS-protected database queries
3. **Database** - PostgreSQL with Row-Level Security policies enforcing data isolation

### Key Patterns

**Frontend Architecture**
- **React Context** for auth state (useAuth hook from AuthContext)
- **View-based components** for full-page flows (LoginView, HomeView, EntryCreationView)
- **Service layer** in `src/services/` abstracts all API calls to backend
- **Card-stack UI** for 4-section entry creation with Framer Motion animations

**Backend Architecture**
- **Better Auth** for authentication (email/password signup/signin)
- **Drizzle ORM** for type-safe database queries
- **Row-Level Security (RLS)** on PostgreSQL tables enforces user data isolation
- **Express middleware** for CORS, error handling, and auth verification

**Database Design**
- **user_profiles** - User metadata (streaks, entry count, onboarding status)
- **entries** - Journal entries with 4 text sections + metadata
  - Unique constraint on `(user_id, entry_date)` - one entry per day max
  - Auto-saves use upsert pattern via `saveEntry()` service
  - RLS policies: Users can only read/write their own entries

### Database Schema

**Key Tables:**
- `user_profiles` - User metadata and tracking
- `entries` - Journal entries with 4 sections (professional_recap, personal_recap, learning_reflections, gratitude)
- `sessions` - Better Auth session management
- `accounts` - Better Auth account linking

**Critical Fields on entries:**
- `entry_date` (DATE) - ISO format `YYYY-MM-DD`, defaults to TODAY()
- `completed` (BOOLEAN) - True when all 4 sections filled
- `created_at`, `updated_at` (TIMESTAMP) - Auto-managed by database
- `user_id` (UUID) - Foreign key, enforced by RLS

### Client Services Layer

**authService.js**
- `signUp(email, password)` - Create new user account
- `signIn(email, password)` - Authenticate user
- `signOut()` - Clear session
- `getCurrentUser()` - Fetch authenticated user

**entryService.js**
- `getEntries()` - Fetch all entries for current user (ordered by date desc)
- `getTodayEntry()` - Get or check for today's entry
- `saveEntry(entryData)` - Upsert entry (creates or updates based on entry_date)
- `deleteEntry(id)` - Delete entry by ID

### Server API Endpoints

**Authentication** (Better Auth)
- `POST /api/auth/sign-up` - Create account
- `POST /api/auth/sign-in` - Login
- `POST /api/auth/sign-out` - Logout

**Entry Operations** (RLS-protected)
- `GET /api/entries` - List user's entries
- `GET /api/entries/:id` - Get specific entry
- `POST /api/entries` - Create or update entry (upsert)
- `DELETE /api/entries/:id` - Delete entry

## Development Guidelines

### Creating Features

1. **Database changes**:
   - Modify schema in `server/src/db/schema.ts`
   - Run `npm run --prefix server db:generate` to create migration
   - Run `npm run --prefix server db:migrate` to apply
   - If new tables: Add RLS policies in `server/src/scripts/apply-rls-prod.ts`

2. **Backend changes**:
   - Add endpoints to `server/src/server.ts` or create new route files
   - Wrap queries with `RLS()` helper to include `user_id` context
   - Test with RLS policies enabled: `npm run --prefix server test:rls`

3. **Frontend changes**:
   - Add service functions in `client/src/services/` first
   - Create components in `client/src/components/` or `views/`
   - Use `useAuth()` hook to access current user
   - Import services and call them from components

### Date Handling

Always use ISO date format `YYYY-MM-DD` for `entry_date`:
```javascript
const today = new Date().toISOString().split('T')[0];
```

### Auto-Save Pattern

Entry creation uses optimistic auto-save:
- Save triggers on section navigation (not on every keystroke)
- Uses `entryService.saveEntry()` with upsert logic
- Shows loading spinner during save
- No explicit "Save" button needed

### RLS (Row-Level Security) Policy

All user data is protected by RLS:
- Tables have `user_id` column with foreign key to `user` table
- `ENABLE RLS` on all user-owned tables
- Policies: Users SELECT/UPDATE/DELETE only rows where `user_id = auth.uid()`
- Admin account (DATABASE_ADMIN_URL) bypasses RLS for migrations
- Test RLS: `npm run --prefix server test:rls`

### Testing

**Multi-layered Testing Strategy**:

```bash
npm run test:all              # Run infrastructure validation + server tests
npm run test:infra            # Infrastructure validation (validates deployment standards)
npm run --prefix server test  # Run Vitest suite (server only)
npm run --prefix server test:rls  # Test Row-Level Security policies
```

**Infrastructure Validation** (`scripts/validate-deployment.ts`):
- Verifies `docker-compose.yml` has no fixed host ports
- Checks that `docker-compose.override.yml` exists and is git-ignored
- Validates Service Worker correctly bypasses `/api` routes
- Ensures `.env.example` keys are consistent

**Server Testing** (Vitest + Supertest):
- Scope: Route protection (401 Unauthorized), health checks, API integrity
- Location: `server/src/__tests__/`
- Framework: Vitest + Supertest for HTTP testing

All tests must pass before merging to main.

### Release Process

Releases are created manually via the **Release workflow** (`.github/workflows/release.yml`).

**Tag format**: `YYYY.MM.DD` for the first release of the day, `YYYY.MM.DD-X` for subsequent ones (e.g. `2026.03.18`, `2026.03.18-2`)

**To create a release**:
1. Go to **Actions → 🚀 Release → Run workflow** on GitHub
2. Optionally add release notes in the input field
3. The workflow automatically:
   - Determines the next sequential tag for today
   - Generates a changelog from commits since the last release
   - Creates the git tag and GitHub release

**Local equivalent**:
```bash
# List existing release tags
git tag -l "release-*" | sort -V

# The workflow handles tag creation — do not create release tags manually
```

### Security & Performance Testing

**Automated Security & Performance Workflow** (`.github/workflows/security-performance.yml`):

Fast, parallel security and performance checks run on every PR. **Optimized execution: 20-30 seconds**.

**Workflow Jobs** (3 parallel checks + report):

1. **🔍 Secret Detection** - 5-10s
   - Scans git history for exposed credentials
   - Blocks PR on failure
   - Configuration: `.gitleaks.toml` (allowlist patterns)

2. **🛡️ Security Scan** - 15-20s
   - SAST scanning with 10 custom rules
   - Blocks on critical vulnerabilities only
   - Configuration: `.semgrep.yml`, `.semgrepignore`

3. **📦 Bundle Size** - 10-15s
   - Tracks client bundle size changes
   - Warns on >10% increase (non-blocking)
   - Baselines: JS 543 KB, CSS 43 KB

4. **📋 Report Generation** - 2-3s
   - Consolidated PR comment with results
   - Action items if issues found
   - Collapsible details for bundle changes

**Report Format:**
- ✅ **All Clear**: Compact success message
- ⚠️ **Warnings**: Recommendations shown
- ❌ **Issues Found**: Actionable checklist with links

**Local Testing:**

```bash
# Secret detection
docker run -v $(pwd):/path zricethezav/gitleaks:latest detect --source=/path --config=.gitleaks.toml

# SAST security scan
docker run --rm -v $(pwd):/src semgrep/semgrep semgrep --config=.semgrep.yml /src

# Bundle size check
cd client && npm run build
find dist/assets -name "*.js" -exec du -ch {} + | grep total
```

**Custom Security Rules** (10 rules in `.semgrep.yml`):
1. Weak encryption keys (client encryption)
2. RLS bypass risk (database queries without wrapper)
3. Missing auth middleware (unprotected API routes)
4. Hardcoded secrets (API keys, passwords)
5. Insecure cookies (missing secure flag)
6. SQL injection (string concatenation in queries)
7. XSS vulnerabilities (unsafe HTML rendering)
8. Missing input validation (API routes)
9. Insecure random (Math.random for security values)
10. Unencrypted storage (sensitive data in localStorage)

**Gitleaks Configuration** (`.gitleaks.toml`):
- Allowlist patterns for docs and test files
- Excludes example credentials (test-*, placeholder-*)
- Ignores workflow test values
- Path-based exclusions for `_bmad-output/`, test directories

**Report Features**:
- Compact table format (easy to scan)
- Expandable bundle details (only when changed)
- Actionable checklist for failures
- Direct links to workflow logs
- Single comment (updates automatically)

**Updating Baselines**:
Edit `.github/workflows/security-performance.yml`:
- Update `JS_BASE=543` and `CSS_BASE=43` variables
- Document reason in PR description

### Styling System

**Frontend Styling** (Tailwind CSS 4.0)
- Custom color palette: `journal-50`, `journal-200`, `journal-500`, `journal-900`, `journal-accent`
- Fonts: Inter (body), Fraunces (headings)
- Framer Motion for animations (card stack transitions, swipe gestures)

### Environment Variables

**Root `.env`**:
```
DATABASE_URL=postgresql://app_user:password@localhost:5432/journal
DATABASE_ADMIN_URL=postgresql://postgres:password@localhost:5432/journal
BETTER_AUTH_SECRET=your-secret-key-min-32-chars
ANTHROPIC_API_KEY=sk-...
RESEND_API_KEY=re_...
```

**Client `.env.local`** (for direct client testing):
```
VITE_API_URL=http://localhost:3000/api
```

**Important**: Always use `http://localhost:5173` (not `127.0.0.1`) for local development to avoid cookie/origin mismatch errors with Better Auth.

## Build Tools

- **Frontend**: Vite with @vitejs/plugin-react (using rolldown-vite for faster builds)
- **Backend**: TypeScript compiled to CommonJS
- **Database**: Drizzle ORM with PostgreSQL driver
- **Auth**: Better Auth (self-hosted, not Supabase in current version)
- **Encryption**: CryptoJS (AES-256) for journal entries at rest
- **Email**: Resend for verification and password resets

## Deployment

**Production Environment**: Self-hosted VPS using Coolify with automatic GitHub webhook integration.

**Domain Strategy**:
- **Production**:
  - Client: `https://journal.philapps.com`
  - API: `https://api.journal.philapps.com`
- **Preview (PR-based)**:
  - Template: `https://{{pr_id}}.journal.philapps.com`
  - API Template: `https://api-{{pr_id}}.journal.philapps.com`
  - Requires wildcard DNS (`*.journal`) pointing to VPS IP

**Critical Deployment Rules**:
1. **Port Mapping**: Never use fixed host port mappings (e.g., `3000:3000`) in `docker-compose.yml`. Coolify manages routing internally. Use `docker-compose.override.yml` (git-ignored) for local development only.
2. **HTTPS Enforcement**: All FQDNs in Coolify must start with `https://`. Better Auth requires HTTPS for secure session cookies.
3. **Proxy Trust**: Express is configured with `app.set('trust proxy', true)` to correctly detect HTTPS behind Coolify's reverse proxy.
4. **Service Worker**: `sw.js` bypasses all `/api` routes. Intercepting API calls breaks authentication, especially during SSL certificate updates.

**Deployment Workflow**:
- Push to `main` branch triggers GitHub webhook
- Webhook automatically deploys to VPS via Coolify
- Coolify handles containerization, build, and service restart
- Environment variables on VPS managed through Coolify dashboard

**Before Merging to Main**:
- Ensure all tests pass: `npm run test:all`
- Run infrastructure validation: `npm run test:infra` (validates docker-compose.yml compliance)
- Test RLS policies: `npm run --prefix server test:rls`
- Code is automatically deployed to production on merge

## Code Quality Standards

- **Linting**: Must pass `npm run lint`. ESLint is configured to allow `motion` prefixes and handle Service Worker globals.
- **Testing**: All changes must pass `npm run test:all` before being merged.
- **Type Checking**: Server-side logic must adhere to TypeScript strict mode.
- **Commit Messages**: Every commit must include a short descriptive summary with bullet-point changelog explaining the "What/Why".

## Development Agent Mandate

Before proposing changes or merging code:
1. Proactively check for linting errors: `npm run lint:client`
2. Run full test suite: `npm run test:all`
3. Run local security checks if modifying sensitive areas (auth, encryption, database):
   - Secret detection: `docker run -v $(pwd):/path zricethezav/gitleaks:latest detect --source=/path`
   - SAST scan: `docker run --rm -v $(pwd):/src semgrep/semgrep semgrep --config=auto /src`
4. Verify Docker deployment impact and `docker-compose.yml` compliance
5. Ensure `.env.example` remains updated with any new variables
6. Maintain `app.set('trust proxy', true)` and Service Worker API bypass in any infrastructure changes
7. Check bundle size impact if adding new client dependencies: `cd client && npm run build`

## Database Backup Safety

**CRITICAL: Always dump to the HOST filesystem, never inside the container.**

```bash
# CORRECT — dump redirected to host
docker exec <pg-container> pg_dump -U postgres journal > ~/journal_backup_$(date +%Y%m%d).sql

# WRONG — /tmp is ephemeral; file is lost when the container is removed
docker exec -it <pg-container> bash -c "pg_dump -U postgres journal > /tmp/backup.sql"
```

**Procedure for PostgreSQL major version upgrades:**

1. Dump to host: `docker exec <pg-container> pg_dump -U postgres journal > ~/journal_backup.sql`
2. Verify file exists and is non-empty on the **host**: `ls -lh ~/journal_backup.sql`
3. Stop stack and remove volume: `docker-compose down -v`
4. Deploy new PG version: `docker-compose up -d --build db`
5. Restore: `docker exec -i <new-pg-container> psql -U postgres journal < ~/journal_backup.sql`
6. Bring up remaining services: `docker-compose up -d`

**Why container `/tmp` is dangerous**: It lives in the container's writable layer and is destroyed when the container is removed. It survives `docker stop` but not `docker rm` or a stack redeploy.

## Important Notes

- **One entry per day**: Database constraint prevents duplicate entries for same user/date
- **RLS is enforced**: Without proper RLS, authentication is worthless; always verify RLS policies are applied
- **Docker is recommended**: Local setup is complex; use Docker for consistent environment with automatic seeding
- **Admin URL for migrations**: `DATABASE_ADMIN_URL` (superuser) needed for schema changes; `DATABASE_URL` (restricted) used by app
- **B-MAD artifacts**: Before making major changes, review the PRD and Architecture in `_bmad-output/planning-artifacts/`
- **Production is automatic**: Merging to main automatically deploys; always test thoroughly before pushing
- **Never modify docker-compose.yml port mappings**: Port mappings break Coolify routing. Use `docker-compose.override.yml` for local development only
- **Origin consistency matters**: Use `http://localhost:5173` to avoid Better Auth cookie/origin issues
- **Seeding is automatic**: The `seed:user` script runs on container start, recreating the test user for consistent credentials

---
> Source: [Philippe-arnd/JournalVibecoded](https://github.com/Philippe-arnd/JournalVibecoded) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
