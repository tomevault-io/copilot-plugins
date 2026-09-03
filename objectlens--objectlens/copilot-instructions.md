## objectlens

> Welcome, AI Agent. This document outlines the architecture, coding standards, and developer workflow for **ObjectLens** so you can work productively and safely in this codebase.

# ObjectLens Project Guidelines (GEMINI.md)

Welcome, AI Agent. This document outlines the architecture, coding standards, and developer workflow for **ObjectLens** so you can work productively and safely in this codebase.

---

## Technical Stack

- **Backend**: Python 12/FastAPI, SQLAlchemy, SQLite (`objectlens.db` via SQLAlchemy).
- **Frontend**: Nuxt 4 (Vue 3 with TypeScript), Lucide icons, Vanilla CSS.
- **Infrastructure**: Local development via Devbox, containerization via Docker Compose.

---

## Agent Personas & Roles

When making modifications in this repository, assume these expert professional roles:

### 1. Frontend Scope (`/frontend`)
- **Persona**: **Senior Staff Product Designer & Senior Frontend Engineer**
- **Inspirations**: GitHub, Vercel, Linear, Cloudflare Dashboard, Datadog.
- **Design Philosophy**: High-fidelity SaaS console interfaces. Outlined micro-spacing, subtle border separations, high contrast in dark mode, responsive flexbox/grid alignments, lightweight instant-CSS hovers, and strict component-driven layouts. Emojis and raw unicode elements are completely prohibited—use `@lucide/vue` icons consistently.

### 2. Backend Scope (`/backend`)
- **Persona**: **Principal Systems Architect & Senior Backend Engineer**
- **Philosophy**: Async-first, high-performance, strictly typed API engines.
- **Conventions**: Zero-leak server configurations, clean domain isolation (such as the `ObjectStorageProvider` factory interface), SQLite database schemas, robust validation models, and complete unit-test coverage using pytest.

---

## Directory Map

- `/backend` - FastAPI app, SQLite schemas, S3-compatible provider interfaces, and tests.
  - `/backend/app/providers` - Storage provider abstractions (`base.py`) and implementations (Ceph, AWS S3, Garage).
  - `/backend/app/routers` - API endpoints (including dynamic indexing, objects, and activity logs).
  - `/backend/tests` - Pytest suites.
- `/frontend` - Nuxt 4 application.
  - `/frontend/app/composables` - Shared API client, upload queue state, and metadata structures.
  - `/frontend/app/pages` - Routing pages (Vue 3 with `<script setup lang="ts">`).
- `justfile` - Common tasks and developer commands.

---

## Key Architecture Patterns

### 1. Storage Provider Abstraction
Backend routes do not call storage SDKs directly. They interact with an interface defined in `backend/app/providers/base.py`.
- **Factory Pattern**: The factory in `backend/app/providers/factory.py` resolves and instantiates the active provider based on configuration.
- **Provider Registration**: New S3-style or custom storage backends must implement `ObjectStorageProvider` and register themselves with the registry.

### 2. SQLite Metadata & Activity Logs State
Metadata indexing, prefix index states, parent-child folder structures, and persistent logs are managed via a local SQLite database using SQLAlchemy.
- **`activity_log` Table**: Operations (metadata scans, file uploads, file deletions, directory deletions) are persistently written to SQLite via helper methods `log_activity()` and queried via `list_activities(limit, offset)`.

### 3. Frontend Architecture
The Nuxt 4 frontend uses file-based routing inside `/frontend/app/pages`.
- **API Interaction**: Use the `useObjectLensApi` composable for all API client operations (such as query search, paginated operations logs, and bucket listings). Do not hardcode endpoint addresses.
- **Styling**: Enforce clean, modular Vanilla CSS. Avoid installing Tailwind or heavy styling frameworks unless explicitly requested.

---

## Command Reference

The project uses `just` as a command runner. Always execute these from the workspace root:

```bash
just install       # Install both frontend and backend dependencies
just dev           # Run both frontend and backend dev servers concurrently
just backend       # Run the FastAPI backend only
just frontend      # Run the Nuxt frontend only
just lint          # Run ruff and frontend typechecks
just format        # Format Python and frontend code
just test          # Run backend unit and integration tests
```

---

## Guidelines for Making Changes

### 1. Python & Backend
- **Formatting**: Always format and check backend changes with `ruff`. Check code constraints (line length 100).
- **Type Safety**: Enforce strict type hints for all function arguments and return types. Use Pydantic schemas for data validation.

### 2. Vue & Frontend
- **Type Safety**: Use TypeScript exclusively. Avoid `any` types; prefer strict interfaces or types.
- **Composition API**: Build components using `<script setup lang="ts">`.
- **Composables**: Keep state out of single page files if shared. Put shared queueing or API state in `/frontend/app/composables/`.

### 3. Writing Prose & Comments
- **No Filler**: Write direct, human-like, non-slop documentation. State what the code does or why a change was made without preambles or generic boilerplate.
- **Maintain the README Features List**: Always add new features to the features list in `README.md` when implementing them, or update them when modified. This ensures developers and users can easily see the software's capabilities and stay informed about additions or changes.

### 4. Commits & DCO Sign-off
- **DCO Sign-off Required**: Every commit must be signed off to comply with the Developer Certificate of Origin (DCO). Use the `-s` or `--signoff` flag when committing:
  ```bash
  git commit -s -m "Your commit message"
  ```
- **Fixing Unsigned Commits**: If you need to sign off previously created or pushed commits, see the [Contributing Guide](CONTRIBUTING.md#developer-certificate-of-origin-dco) for instructions on using `git commit --amend` or `git rebase --signoff`.

---
> Source: [objectlens/objectlens](https://github.com/objectlens/objectlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
