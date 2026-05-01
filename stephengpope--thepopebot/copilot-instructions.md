## thepopebot

> Technical reference for AI assistants modifying the thepopebot NPM package source code.

# thepopebot — Package Source Reference

Technical reference for AI assistants modifying the thepopebot NPM package source code.

**Architecture**: Event Handler (Next.js) creates `agent-job/*` branches → launches Docker agent container locally (Claude Code, Pi, etc.) → task executed → PR created → auto-merge → notification. Agent jobs log to `logs/{JOB_ID}/`.

## Deployment Model

The npm package (`api/`, `lib/`, `config/`, `bin/`) is published to npm. In production:

- **Event handler**: Docker image bakes the npm package, Next.js app source (`web/`), and `.next` build output. User project directories (`agent-job/`, `event-handler/`, `skills/`, `.env`, `data/`, etc.) are individually volume-mounted into `/app`. The full project is also mounted at `/project` for git access. Runs `server.js` via PM2 behind Traefik reverse proxy.
- **`lib/paths.js`**: Exports `PROJECT_ROOT` (`process.cwd()`). This is how the installed npm package finds the volume-mounted user project files.
- **Agent-job containers**: Ephemeral Docker containers clone `agent-job/*` branches separately — use named volumes for workspace. See `docker/CLAUDE.md`.
- **Local install**: Gives users CLI tools (`init`, `setup`, `upgrade`) and configuration scaffolding.

## Package vs. Templates — Where Code Goes

All event handler logic, API routes, library code, and core functionality lives in the **npm package** (`api/`, `lib/`, `config/`, `bin/`). This is what users import when they `import ... from 'thepopebot/...'`.

The `templates/` directory contains **only files that get scaffolded into user projects** via `npx thepopebot init`. Templates are for user-editable configuration and thin wiring — things users are expected to customize or override. Never add core logic to templates.

**When adding or modifying event handler code, always put it in the package itself (e.g., `api/`, `lib/`), not in `templates/`.** Templates should only contain:
- Configuration files users edit (`agent-job/SOUL.md`, `agent-job/CRONS.json`, `event-handler/TRIGGERS.json`, etc.)
- GitHub Actions workflows
- Docker compose (`docker-compose.yml`)
- CLAUDE.md files for AI assistant context in user projects

Next.js app source files (`app/`, `next.config.mjs`, `server.js`, etc.) live in `web/` at the package root. These are built into the Docker image — NOT scaffolded to user projects.

### Managed Paths

Files in managed directories are auto-synced (created, updated, **and deleted**) by `init` to match the package templates exactly. Users should not edit these files — changes will be overwritten on upgrade. Managed paths are defined in `bin/managed-paths.js`:

- `.github/workflows/` — CI/CD workflows
- `docker-compose.yml`, `.dockerignore` — Docker config
- `.gitignore` — Git ignore rules
- `CLAUDE.md` — AI assistant context

## Directory Structure

```
/
├── api/                        # GET/POST handlers for all /api/* routes
├── lib/
│   ├── actions.js              # Shared action executor (agent, command, webhook)
│   ├── cron.js                 # Cron scheduler (loads CRONS.json)
│   ├── triggers.js             # Webhook trigger middleware (loads TRIGGERS.json)
│   ├── paths.js                # Exports PROJECT_ROOT (process.cwd())
│   ├── ai/                     # LLM integration (agent, model, tools, streaming)
│   ├── auth/                   # NextAuth config, helpers, middleware, server actions, components
│   ├── channels/               # Channel adapters (base class, Telegram, factory)
│   ├── chat/                   # Chat route handler, server actions, React UI components
│   ├── cluster/                # Worker clusters (roles, triggers, Docker containers)
│   ├── code/                   # Code workspaces (server actions, terminal view, WebSocket proxy)
│   ├── containers/             # Container SSE streaming (Docker container status)
│   ├── db/                     # SQLite via Drizzle (schema, migrations, api-keys)
│   ├── tools/                  # Job creation, GitHub API, Telegram, Docker, Whisper
│   ├── voice/                  # Voice input (AssemblyAI streaming transcription)
│   └── utils/
│       └── render-md.js        # Markdown {{include}} processor
├── config/
│   ├── index.js                # withThepopebot() Next.js config wrapper
│   └── instrumentation.js      # Server startup hook (loads .env, starts crons)
├── bin/                        # CLI entry point (init, setup, reset, diff, upgrade)
├── setup/                      # Interactive setup wizard
├── web/                        # Next.js app source (baked into Docker image, NOT scaffolded)
│   ├── app/                    # Next.js app directory (pages, layouts, routes)
│   ├── server.js               # Custom Next.js server with WebSocket proxy
│   ├── next.config.mjs         # Next.js config wrapper
│   ├── instrumentation.js      # Server startup hook
│   ├── middleware.js            # Auth middleware
│   └── postcss.config.mjs      # PostCSS/Tailwind config
├── templates/                  # Scaffolded to user projects (see rule above)
├── docs/                       # Extended documentation
└── package.json
```

## NPM Package Exports

Exports defined in `package.json` `exports` field. Pattern: `thepopebot/{module}` maps to source files in `api/`, `lib/`, `config/`. Includes `./cluster/*`, `./voice/*` exports. Add new exports there when creating new importable modules.

## UI Component Standards

Settings/admin pages use shared components from `lib/chat/components/settings-shared.jsx`. See `lib/chat/components/CLAUDE.md` for the full UI standards (button tiers, dialogs, save feedback, delete confirmation, spacing, etc.). **Follow these standards when adding new settings pages.**

## Build System

Run `npm run build` before publish. esbuild compiles `lib/chat/components/**/*.jsx`, `lib/auth/components/**/*.jsx`, `lib/code/*.jsx`, `lib/cluster/components/**/*.jsx` to ES modules.

## Database

SQLite via Drizzle ORM at `data/thepopebot.sqlite` (override with `DATABASE_PATH`). Auto-initialized on server start. See `lib/db/CLAUDE.md` for schema details, CRUD patterns, and column naming.

### Migration Rules

**All schema changes MUST go through the migration workflow.**

- **NEVER** write raw `CREATE TABLE`, `ALTER TABLE`, or any DDL SQL manually
- **NEVER** modify `initDatabase()` to add schema changes
- **ALWAYS** make schema changes by editing `lib/db/schema.js` then running `npm run db:generate`

## Security: Route Architecture

**`/api` routes are for external callers only.** They authenticate via `x-api-key` header or webhook secrets (Telegram, GitHub). Never add session/cookie auth to `/api` routes.

**Browser UI uses fetch route handlers colocated with pages.** All authenticated browser-to-server calls use Next.js route handlers (`route.js` files in `web/app/`) that check `auth()` session. Do NOT use server actions for data fetching — they cause page refresh issues. Handler implementations live in `lib/chat/api.js`; route files are thin re-exports.

**`/stream/*` is for actual SSE streaming only.** Three endpoints use Server-Sent Events: `/stream/chat` (AI SDK streaming), `/stream/containers` (Docker container status), `/stream/cluster/[clusterId]/logs` (cluster logs). All other fetch routes are colocated with their page directories.

| Caller | Mechanism | Auth | Location |
|--------|-----------|------|----------|
| External (cURL, GitHub Actions, Telegram) | `/api` route handler | `x-api-key` or webhook secret | `api/index.js` |
| Browser UI (data/mutations) | Fetch route handler colocated with page | `auth()` session check | `web/app/<page>/route.js` |
| Browser UI (SSE streaming) | EventSource / AI SDK streaming | `auth()` session check | `web/app/stream/` |

## Action Dispatch System

Shared executor for cron jobs and webhook triggers (`lib/actions.js`). Three action types: `agent` (Docker LLM container), `command` (shell command), `webhook` (HTTP request). See `lib/CLAUDE.md` for detailed dispatch format, cron/trigger config, and template tokens.

## LLM Providers

See `lib/ai/CLAUDE.md` for the provider table and model defaults. Key: `LLM_PROVIDER` + `LLM_MODEL` env vars, `LLM_MAX_TOKENS` defaults to 4096.

## Workspaces

- **Code Workspaces**: Interactive Docker containers with in-browser terminal. See `lib/code/CLAUDE.md`.
- **Cluster Workspaces**: Groups of Docker containers spawned from role definitions with triggers. See `lib/cluster/CLAUDE.md`.

Both use `lib/tools/docker.js` for container lifecycle via Unix socket API.

## Skills System

Skills live in `skills/library/`. Activate by symlinking into `skills/active/`. Each skill has `SKILL.md` with YAML frontmatter (`name`, `description`). The `{{skills}}` template variable in markdown files resolves active skill descriptions at runtime. Default active skill: `get-secret`. Pi agent auto-activates `browser-tools` (other agents use Playwright MCP).

## Template Config & Markdown Includes

Config markdown files support `{{ filepath.md }}` includes (resolved relative to project root) and built-in variables (`{{datetime}}`, `{{skills}}`), powered by `lib/utils/render-md.js`.

## Config Variable Architecture

`LLM_MODEL` and `LLM_PROVIDER` exist in two separate systems using the same names:

- **`.env`** — read by the event handler (chat). Set by `setup/lib/sync.mjs`.
- **GitHub repository variables** — read by agent job containers. Set by `setup/lib/sync.mjs`.

These are independent environments. They use the same variable names. They can hold different values (e.g. chat uses sonnet, jobs use opus). Do NOT create separate `AGENT_LLM_*` variable names — just set different values in `.env` vs GitHub variables.

---
> Source: [stephengpope/thepopebot](https://github.com/stephengpope/thepopebot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
