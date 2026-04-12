## movienight

> A full-stack movie suggestion app: React + TypeScript frontend, Apollo GraphQL backend, PostgreSQL database. Containerized with Docker Compose.

# MovieNight — Agent Context

A full-stack movie suggestion app: React + TypeScript frontend, Apollo GraphQL backend, PostgreSQL database. Containerized with Docker Compose.

## Quick orientation

| Layer | Location | Key files |
|---|---|---|
| Frontend | `src/` | `App.tsx`, `src/components/`, `src/graphql/queries.ts`, `src/contexts/AuthContext.tsx` |
| Backend | `backend/src/` | `index.ts`, `schema.ts`, `resolvers.ts`, `db.ts`, `auth.ts`, `scheduler.ts` |
| DB migrations | `backend/migrations/` | numbered JS files run by node-pg-migrate |
| Docker | root | `docker-compose.yml`, `Dockerfile`, `nginx.conf` |
| Backend Docker | `backend/` | `backend/Dockerfile` |

## Running the project

```bash
# Start everything (recommended)
docker-compose up -d
# Frontend → http://localhost:3000
# GraphQL  → http://localhost:4000/graphql
# Postgres → localhost:5432

# Local dev without Docker
npm install && npm start            # frontend
cd backend && npm install && npm run dev  # backend (needs Postgres separately)

# Build
npm run build                       # frontend (CRA)
cd backend && npm run build         # backend (tsc)

# Migrations (run automatically on backend startup)
cd backend && npm run migrate:up
cd backend && npm run migrate:down
cd backend && npm run migrate:create <name>
```

## Architecture

- **Frontend**: React 18 + TypeScript, Apollo Client 3, MUI Joy, Styled Components, @dnd-kit (drag-and-drop)
- **Backend**: Node 18/20, Apollo Server 4 + Express, pg (no ORM), bcrypt + JWT
- **Database**: PostgreSQL 15
- **Auth**: JWT (7-day expiry) stored in localStorage, sent as `Authorization: Bearer <token>`. Context injected per-request in `backend/src/index.ts`.
- **Ranking**: Movies ordered by `rank NUMERIC(20,10)` using fractional indexing; new movies appended at `MAX(rank) + 1`.
- **Polling**: `GET_MOVIES` query polls every 5 s (`pollInterval: 5000`).
- **Production gating**: Kometa export/scheduling and certain mutations are blocked unless `NODE_ENV === 'production'`. `appInfo.isProduction` query exposes this to the frontend.

## GraphQL schema

```graphql
# Queries
appInfo: AppInfo                                   # isProduction flag
movies: [Movie!]!                                  # unwatched only (watched_at IS NULL), ordered by rank
movie(id: ID!): Movie
me: User                                           # requires auth
users: [User!]!                                    # admin only
user(id: ID!): User                                # admin only
searchTmdb(query: String!): [TmdbMovie!]!          # TMDB search (needs TMDB_API_KEY)
auditLogs(limit: Int, offset: Int): [AuditLog!]!  # admin only, max 500
loginHistory(userId: ID, limit: Int): [LoginHistory!]!  # admin only, max 500
kometaSchedule: KometaSchedule                     # admin only

# Mutations
addMovie(title: String!, tmdb_id: Int): Movie!         # requires auth
matchMovie(id: ID!, tmdb_id: Int!, title: String!): Movie!  # requires auth, owner or admin
markWatched(id: ID!): Movie!                            # requires auth, owner or admin
deleteMovie(id: ID!): Boolean!                          # admin only
reorderMovie(id: ID!, afterId: ID): Movie!             # admin only (afterId=null → move to top)
exportKometa(collectionName: String): KometaExportResult!  # admin + production only
updateKometaSchedule(...): KometaSchedule!             # admin + production only
importFromLetterboxd(url: String!): ImportResult!      # admin only
login(username: String!, password: String!): AuthPayload!
createUser / updateUser / deleteUser                   # admin only
```

All GraphQL operations are defined in `src/graphql/queries.ts`.

## Component structure

```
src/components/
├── admin/
│   ├── AdminPanel.tsx       # tabs: Users, Audit Logs, Login History, Kometa, Letterboxd
│   ├── UserManagement.tsx   # CRUD UI for users
│   ├── AuditLog.tsx         # paginated audit log viewer
│   ├── LoginHistory.tsx     # login attempts with IP/user-agent
│   ├── KometaExport.tsx     # manual export + scheduler config UI
│   └── LetterboxdImport.tsx # URL input form, shows imported/skipped/tmdb_matched counts
├── auth/
│   └── Login.tsx            # username/password form
├── common/
│   ├── Navbar.tsx           # Movies/Admin navigation, Logout
│   └── Footer.tsx           # deploy timestamp, git branch/hash
└── home/
    ├── Homepage.tsx          # movie list, drag-to-reorder (@dnd-kit), Add Movie, Mark Watched
    └── TmdbMatchFlow.tsx     # TMDB search + match UI for unmatched movies
```

## Key conventions

- **No ORM** — raw SQL via `pg` pool (`backend/src/db.ts`). Use parameterised queries (`$1, $2, …`).
- **Authorization** — check `context.user` (authenticated) or `context.user?.isAdmin` (admin) in resolvers; throw `GraphQLError` with appropriate `extensions.code`.
- **Field resolvers** — timestamps from Postgres are converted to ISO 8601 in `Movie.date_submitted`, `Movie.watched_at`, `User.created_at/updated_at/last_login_at`, `AuditLog.created_at`, `LoginHistory.created_at`.
- **Movie.requester** field resolved from `requested_by` FK (display_name or username), falls back to `'Unknown'`.
- **Frontend state** — auth state lives in `AuthContext`; movie/user data comes from Apollo cache.
- **Env vars** — frontend uses `REACT_APP_*`; backend uses bare names.

## Database schema (current)

| Table | Notable columns |
|---|---|
| `movies` | id, title, requester (text), requested_by (FK→users), date_submitted, rank (NUMERIC 20,10), tmdb_id (nullable int), watched_at (nullable timestamptz) |
| `users` | id, username, password_hash, email, display_name, is_admin, is_active, last_login_at, created_at, updated_at |
| `audit_logs` | id, actor_id (FK→users), action, target_type, target_id, metadata (JSONB), ip_address, created_at |
| `login_history` | id, user_id (FK→users), ip_address, user_agent, succeeded, created_at |
| `kometa_schedule` | id, enabled, frequency, daily_time, collection_name, last_run_at, updated_at |

## Audit log actions

`MOVIE_ADD`, `MOVIE_WATCHED`, `MOVIE_DELETE`, `MOVIE_REORDER`, `MOVIE_TMDB_MATCH`, `LOGIN_SUCCESS`, `LOGIN_FAILURE`, `USER_CREATE`, `USER_UPDATE`, `USER_DELETE`, `KOMETA_EXPORT`, `KOMETA_SCHEDULE_EXPORT`, `LETTERBOXD_IMPORT`

## Key features

### Watched tracking
`markWatched(id)` sets `watched_at = NOW()`. `movies` query returns only `WHERE watched_at IS NULL`.

### Drag-and-drop reordering
`reorderMovie(id, afterId?)` uses fractional indexing (midpoint between neighbors). `afterId=null` moves movie to top (`rank / 2`). Frontend uses `@dnd-kit/core` + `@dnd-kit/sortable`.

### TMDB integration
`searchTmdb(query)` hits TMDB API (requires `TMDB_API_KEY`). `matchMovie(id, tmdb_id, title)` links a movie to its TMDB entry. `addMovie` accepts optional `tmdb_id`.

### Kometa/Plex integration (production-only)
`exportKometa` writes a YAML collection file to `KOMETA_COLLECTIONS_PATH` and optionally POSTs to `KOMETA_TRIGGER_URL`. `scheduler.ts` runs automated exports (hourly or daily); `initScheduler()` called on startup only when `NODE_ENV === 'production'`.

### Letterboxd import
`importFromLetterboxd(url)` fetches the public Letterboxd list, parses film titles from HTML (data-item-name), skips duplicates (case-insensitive), optionally matches to TMDB. Returns `{ imported, skipped, tmdb_matched, errors }`.

## Environment variables

**Frontend** (`.env`):
- `REACT_APP_GRAPHQL_URL` — defaults to `/graphql`
- `REACT_APP_GIT_BRANCH`, `REACT_APP_GIT_HASH`, `REACT_APP_DEPLOY_TIME` — baked in at Docker build time

**Backend** (`backend/.env`):
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`
- `PORT` (default 4000)
- `JWT_SECRET` — **change in production** (default: `'your-secret-key-change-in-production'`)
- `ADMIN_PASSWORD` — seeds the default admin user (default `admin123`, **change in production**)
- `NODE_ENV` — `'production'` enables Kometa features; anything else disables them
- `TMDB_API_KEY` — optional; enables TMDB search and Letterboxd TMDB matching
- `KOMETA_COLLECTIONS_PATH` — directory for exported Kometa YAML files
- `KOMETA_TRIGGER_URL` — optional webhook URL to trigger Kometa after export

## Docker profiles

| Profile | Services |
|---|---|
| `development` | db (postgres:15-alpine), backend, frontend (hot-reload) |
| `production` | Single `movienight` container (ghcr.io image, port 8080→80, 1 CPU / 512 MB limit) |

**Production image**: Multi-stage Dockerfile — node:20-alpine builds React, nginx:alpine serves static files + proxies `/graphql` to `movienight-backend:4000`.

## CI/CD

- `dev` branch push → `.github/workflows/test-build.yml` — build only (no push), with dorny/paths-filter to skip unchanged layers
- `master` branch push → `.github/workflows/deploy.yml` — build + push to GHCR, SSH deploy to remote host

## Existing docs

- `README.md` — quick-start and API examples
- `AUTHENTICATION.md` — JWT flow, default credentials, user management
- `DEPLOYMENT.md` — CI/CD setup, SSH keys, GHCR config
- `backend/MIGRATIONS.md` — migration conventions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nova-firefly)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nova-firefly)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
