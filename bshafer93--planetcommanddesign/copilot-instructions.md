## planetcommanddesign

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Planet Command Design** is a game design tools & calculators site for the "Planet Command" universe. It provides interactive tools for spacecraft engineering: struct memory sizing, power generation design, production chain logistics, and physics calculations (projectiles, lasers, particle beams).

## Build & Dev Commands

```bash
npm run dev          # Runs Vite dev server + Express backend concurrently
npm run dev:client   # Vite dev server only
npm run dev:server   # Express backend only (tsx watch)
npm run build        # svelte-check && vite build
npm run preview      # Preview production build
```

The dev server proxies `/api` requests to `http://localhost:3001` (Express backend).

There is no test runner or linter configured. `npm run build` runs `svelte-check` for type-checking before the Vite build.

## Directory Structure

```
├── .github/workflows/deploy.yml   # CI/CD: auto-deploy on push to main
├── server/                         # Express backend (port 3001)
│   ├── index.ts                    # Entry point, CORS, JSON middleware
│   ├── routes/save.ts              # POST /api/save — JSON file persistence
│   ├── tsconfig.json               # Backend TS config (ES2022, Node)
│   └── data/                       # Runtime JSON storage (gitignored)
├── src/
│   ├── main.ts                     # Svelte app mount point
│   ├── App.svelte                  # Root layout, tab bar, hash routing
│   ├── types.ts                    # ToolDef interface
│   ├── styles/global.css           # Dark theme CSS custom properties
│   ├── components/
│   │   └── SliderInput.svelte      # Reusable slider+number input (linear/log)
│   └── tools/
│       ├── struct-sizer/           # C++ struct memory calculator
│       ├── power-gen/              # Power generation & thruster design
│       ├── production-chain/       # Spacecraft construction logistics
│       └── calculations/           # Physics calculators (projectile/laser/particle beam)
├── index.html                      # HTML entry point
├── package.json                    # ES module, scripts, deps
├── tsconfig.json                   # Frontend TS config (ES2020, DOM)
├── vite.config.ts                  # Vite + Svelte plugin, /api proxy
└── svelte.config.js                # Svelte preprocessor config
```

## Architecture

**Svelte 5 + Vite + TypeScript** game design tools site with tab-based navigation. ES modules throughout (`"type": "module"` in package.json).

### Frontend

- **Hash routing** — `App.svelte` uses `window.location.hash` (#tool-id) for tab navigation, no router library. Components mount/unmount via `{#key activeToolId}`.
- **Tool registration** — Each tool is a Svelte component in `src/tools/<name>/`. Register by importing the component and adding `{ id, label, component }` to the `tools` array in `App.svelte`. The `ToolDef` interface is in `src/types.ts`.
- **Svelte 5 runes** — Use `$state()`, `$derived()`, `$derived.by()`, `$effect()`, `$props()`, `$bindable()`, `bind:value`, `onclick={handler}`, and direct component references (not `svelte:component`).
- **Styling** — Dark theme via CSS custom properties in `src/styles/global.css`. Fonts: "DM Sans" (body), "JetBrains Mono" (headers/code). No CSS framework. Scoped `<style>` blocks in each component.
- **Charts** — Chart.js v4 with canvas refs (`bind:this`), plus `chartjs-plugin-annotation` for threshold/reference lines. Colors are hardcoded hex constants (CSS vars don't work in canvas). Charts reuse instances via `.update('none')` instead of destroy/recreate for performance. Raw canvas 2D API used for heatmaps (e.g., range-vs-power in laser calculator, emittance feasibility, particle beam heatmaps).
- **Viz panels** — Each chart/visualization is wrapped in a `<details class="viz-toggle">` for collapsible show/hide, with HTML5 drag-and-drop reordering via CSS `order` property (keeps canvas bindings intact since DOM nodes don't move).
- **Reusable components** — Shared UI components live in `src/components/`. Use `SliderInput` for any slider+number input combo (supports linear and `log` mode for large ranges, with optional `inputMax` override).

### Adding a New Tool

1. Create `src/tools/<tool-id>/ToolName.svelte`
2. Optionally add a `data/` subdirectory for JSON reference data
3. Import the component in `App.svelte`
4. Add `{ id: '<tool-id>', label: '<tab_label>', component: ToolName }` to the `tools` array
5. The tool is now routable at `#<tool-id>`

### Current Tools

- **struct-sizer** (`src/tools/struct-sizer/`) — C++ struct memory/padding estimator. Supports 51 types (primitives, STL containers, pointers). No external data files.
- **power-gen** (`src/tools/power-gen/`) — Power generator & thruster design. Sub-tabs for Generators (4 charts) and Thrusters (placeholder). Reference data in `data/*.json` (10 files: generator-comparison, fuel-energy-density, radiator-cooling, radiator-materials, carrier-power-bounds, nuclear-conversion-chain, water-use-nuclear, direct-conversion-methods, fusion-size-factors, fusion-tech-improvements).
- **production-chain** (`src/tools/production-chain/`) — Production chain design for spacecraft construction. Reference data in `data/*.json` (4 files: material-breakdown, raw-resource-requirements, production-constraints, logistics-vehicles). Charts: material doughnut breakdowns, raw resource bars, raw-vs-finished stacked bars, hauls-to-build log-scale comparisons. Two reference ship scales: 30t baseline orbital-class and 100kt carrier.
- **calculations** (`src/tools/calculations/`) — Physics calculators with sub-tabs: Projectile (KE = ½mv², shape/material selectors from materials DB), Laser (penetration via energy-balance melt-through model with slider inputs, penetration bar + rate, time-vs-depth chart, range-vs-power heatmap), and Particle Beam (combat tab with manual gradient slider + auto-optimal reference, emittance line chart + feasibility heatmap, accelerator design tab with spot-vs-range chart and 3D Plotly surface). Laser and Particle Beam share a target material selector and adjustable plate thickness (1–200 cm) from the materials database; absorption model remains steel-specific. Both display penetration rate (mm/s, cm/s). Emittance is evaluated at source beam diameter (accelerator exit), not at target; range calculations show spot expansion and power delivery at distance. Min beam diameter: 5mm. Data in `data/*.json` (7 files: materials-database with 82 materials, particle-beam-feasibility, particle-beam-comparison, particle-beam-divergence, particle-beam-efficiencies, particle-beam-target, particle-beam-physics-limits).

### CSS Custom Properties

Defined in `src/styles/global.css` on `:root`:

| Variable        | Value                          | Usage          |
|-----------------|--------------------------------|----------------|
| `--bg`          | `#0e1117`                      | Page background |
| `--surface`     | `#161b22`                      | Card/panel bg  |
| `--surface2`    | `#1c2231`                      | Nested panel bg |
| `--border`      | `#2a3142`                      | Borders        |
| `--text`        | `#e2e8f0`                      | Primary text   |
| `--text-dim`    | `#8492a6`                      | Secondary text |
| `--accent`      | `#60a5fa`                      | Blue accent    |
| `--accent-glow` | `rgba(96, 165, 250, 0.15)`    | Glow effects   |
| `--red`         | `#f87171`                      | Red            |
| `--green`       | `#4ade80`                      | Green          |
| `--yellow`      | `#fbbf24`                      | Yellow         |
| `--purple`      | `#c084fc`                      | Purple         |
| `--orange`      | `#fb923c`                      | Orange         |
| `--pink`        | `#f472b6`                      | Pink           |
| `--cyan`        | `#22d3ee`                      | Cyan           |
| `--teal`        | `#2dd4bf`                      | Teal           |

**Note:** CSS custom properties cannot be used inside `<canvas>` elements. Use hardcoded hex values matching the above when drawing on Chart.js or 2D canvas.

### Backend

Express server in `server/` on port 3001. Entry: `server/index.ts`, routes in `server/routes/`. JSON file storage in `server/data/` (gitignored). Has its own `tsconfig.json` targeting ES2022/Node.

Endpoints:
- `GET /api/health` — returns `{ status: "ok" }`, used by Docker health checks
- `POST /api/save` — accepts `{ filename: string, data: any }`, sanitizes filename (alphanumeric/dash/underscore only), writes JSON to `server/data/`.
- `GET /api/orbit/planets` — list available planet names for porkchop plots
- `POST /api/orbit/porkchop` — compute porkchop plot grid (Lambert solver, JPL Horizons ephemeris)

### Data File Conventions

Tool-specific reference data lives in `src/tools/<name>/data/*.json`. These are static JSON files imported at build time. Common patterns:
- Top-level `title` and `description` fields for documentation
- Arrays of objects with numeric properties for chart data
- Named keys for lookup tables (e.g., materials, technologies)

### Docker Compose (Full Stack)

The app runs as a three-service Docker Compose stack, managed by **Komodo** on the Docker VM:

| Service | Base Image | Role |
|---------|------------|------|
| `frontend` | `caddy:2-alpine` | Caddy serving built Svelte SPA, TLS termination, proxies `/api/*` to backend, HTTP→HTTPS redirect |
| `backend` | `node:20-alpine` | Express API (port 3001), connects to Postgres via `DATABASE_URL` |
| `db` | `postgres:latest` | PostgreSQL database |

**Multi-stage Dockerfile** (`Dockerfile`):
- Stage `build`: Node 20 Alpine, runs `vite build`
- Stage `frontend`: Caddy 2 Alpine, copies built assets to `/srv` + `Caddyfile`
- Stage `backend`: Node 20 Alpine, copies `server/` source, runs via `tsx`

**Caddyfile** — Serves static files from `/srv`, proxies `/api/*` to `backend:3001`, TLS with mkcert certs mounted at `/certs`, HTTP→HTTPS redirect. Uses `auto_https disable_redirects` and `try_files {path} /index.html` for SPA routing.

**Networking** — Two isolated Docker networks:
- `frontend` network: frontend (Caddy) + backend (Caddy proxies `/api/*` to Express)
- `backend` network: backend + db (Express connects to Postgres)
- Frontend container cannot directly reach the database

**Health checks & startup order:**
- `db`: `pg_isready` check, backend waits for `service_healthy`
- `backend`: `wget http://127.0.0.1:3001/api/health` check, frontend waits for `service_healthy`
- **Important:** Use `127.0.0.1`, NOT `localhost` — Alpine wget resolves `localhost` to IPv6 `::1` but Node listens on IPv4 `0.0.0.0`

**Volumes:**
- `pgdata` — PostgreSQL data at `/var/lib/postgresql` (pg 18+ compatible mount point)
- `app-data` — JSON file storage for `/api/save`
- `caddy-data` — Caddy internal state
- `/mnt/truenas/planet-command/certs` — mkcert TLS certs (read-only bind mount)

**Environment** (`.env` file, gitignored):
- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` — used by Postgres container
- `DATABASE_URL` — connection string for backend (`postgresql://user:pass@db:5432/dbname`)

**Ports:**
- Frontend (Caddy): `192.168.100.52:443` and `192.168.100.52:80` (HTTPS + HTTP redirect)
- Database (Postgres): `192.168.100.52:5432` (external access for DBeaver/pgAdmin)

**Build:** Images are built locally by Komodo (no registry pull needed). `docker compose build` for local builds.

### Database

**PostgreSQL** (postgres:latest / v18) running as `db` service in the compose stack.

| Field | Value |
|-------|-------|
| Host | `192.168.100.52` (external) / `db` (internal Docker network) |
| Port | `5432` |
| Database | `PlanetCommandDesignDB` |
| Username | `planetcommand` |
| Password | `planetcommand_db_pass` |
| Connection string | `postgresql://planetcommand:planetcommand_db_pass@db:5432/PlanetCommandDesignDB` |

- Credentials are in `.env` (gitignored): `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `DATABASE_URL`
- Backend connects via `DATABASE_URL` environment variable
- Data persisted in `pgdata` Docker volume at `/var/lib/postgresql`
- External port bound to VLAN 100 IP (`192.168.100.52:5432`) for DBeaver/pgAdmin access

### Deployment

**Komodo (primary)** — Manages the Docker Compose stack on the Docker VM (192.168.1.50):
- Pulls from `bshafer93/PlanetCommandDesign` `main` branch
- Builds images locally, runs `docker compose up`
- Komodo URL: `https://komodo.docker.home`
- Stack accessible at `https://planetcommand.home` (DNS → 192.168.100.52)
- Stack ID: `6994a98204c61f7c00e17bde`
- API auth: `POST https://komodo.docker.home/auth` with `{"type":"LoginLocalUser","params":{"username":"admin","password":"bKX6u6jBkHtnHtEFw9oySw"}}` → JWT
- API calls: `POST /read`, `/write`, `/execute` with `authorization: <jwt>` header (no `Bearer` prefix)
- Deploy: `POST /execute` with `{"type":"DeployStack","params":{"stack":"6994a98204c61f7c00e17bde"}}`

**GitHub Actions (legacy/CI)** — `.github/workflows/deploy.yml` auto-deploys on push to `main`:
- Builds with Node 20 (`npm ci && npm run build`)
- Rsyncs to DigitalOcean droplet (separate from Docker VM deployment)
- Secrets: `DEPLOY_SSH_KEY`, `DEPLOY_HOST`

### Key Dependencies

| Package               | Version  | Purpose                                |
|-----------------------|----------|----------------------------------------|
| svelte                | ^5.0.0   | UI framework (runes API)               |
| vite                  | ^6.1.0   | Bundler and dev server                 |
| chart.js              | ^4.5.1   | Charts (bar, doughnut, line, scatter)  |
| chartjs-plugin-annotation | ^3.1.0 | Threshold/reference lines on charts  |
| plotly.js-gl3d-dist-min | ^3.0.0 | 3D surface plots (particle beam)       |
| express               | ^4.21.0  | Backend API server                     |
| typescript            | ^5.7.0   | Type checking                          |
| tsx                   | ^4.19.0  | TypeScript execution for backend       |

### NASA Ephemeris Data (`nasa_data/`)

Download scripts for NASA JPL DE441 planetary/lunar ephemeris data (~85 GB total). Two equivalent scripts:

- `nasa_data/download_de441.sh` — Bash version (curl-based)
- `nasa_data/download_de441.ps1` — PowerShell version (.NET HttpWebRequest)

Both support resume (safe to interrupt and re-run), auto-retry (5 attempts), and skip already-complete files. Toggle `$Download*` / `DOWNLOAD_*` flags at the top to enable/disable categories.

**Data categories:**

| Category | Directory | Size | Source |
|----------|-----------|------|--------|
| BSP (SPICE kernels) | `nasa_data/bsp/` | ~3.2 GB | `ssd.jpl.nasa.gov/ftp/eph/planets/bsp/` |
| ASCII coefficients | `nasa_data/ascii/` | ~8.8 GB | 31 Chebyshev files (-13000 to +17000) |
| Linux binary | `nasa_data/linux/` | ~2.6 GB | Native binary format |
| NIO format | `nasa_data/nio/` | ~3.3 GB | Network-independent format |
| Small bodies | `nasa_data/small_bodies/` | ~15.7 GB | 373 asteroid perturbers for DE441 |
| Satellites | `nasa_data/satellites/` | ~50 GB | Moon ephemerides (all planets + TNOs) |
| Lunar frames | `nasa_data/bpc/` | ~13 MB | Orientation frame kernels |
| Documentation | `nasa_data/docs/` | ~30 MB | IOMs, papers |
| Fortran readers | `nasa_data/fortran/` | ~100 KB | `asc2eph.f`, `testeph.f` |
| Test data | `nasa_data/test_data/` | ~10 MB | Verification test files |

**Run:**
```powershell
cd nasa_data
.\download_de441.ps1                                          # PowerShell
# or
powershell -ExecutionPolicy Bypass -File .\download_de441.ps1  # if policy restricted
```
```bash
cd nasa_data && chmod +x download_de441.sh && ./download_de441.sh  # Bash
```

**Note:** `nasa_data/` contents should be gitignored — these are large binary/ASCII data files downloaded from JPL.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bshafer93) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
