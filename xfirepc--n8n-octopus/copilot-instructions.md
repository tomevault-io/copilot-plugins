## n8n-octopus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Octopus is an orchestration system for managing multiple N8N workflow automation instances with shared PostgreSQL, Redis, and a centralized Nginx reverse proxy. All infrastructure runs as Docker containers on a shared `n8nmanager-network` bridge network.

The project is written in Spanish (UI, docs, variable names in some places).

## Commands

```bash
# Development (runs backend + frontend concurrently)
pnpm dev

# Run backend only (with --watch auto-reload)
cd app/backend && pnpm dev

# Run frontend only (Vite dev server on :5173)
cd app/frontend && pnpm dev

# Install all dependencies
pnpm install:all

# Build frontend for production
pnpm build

# Lint frontend
cd app/frontend && pnpm lint
```

Tests are not implemented yet (scripts are stubs).

## Architecture

**Monorepo** using pnpm workspaces with two packages: `app/backend` and `app/frontend`. ES Modules throughout (`"type": "module"`).

### Backend (Express.js, port 3000)

```
app/backend/src/
├── index.js              # Entry point, Express setup, CORS, BackupScheduler init
├── routes/api.js         # All API routes (/api/*), auth middleware applied here
├── controllers/          # Request handlers (sites, docker, ssl, system, databases, backups, nginx, templates, auth)
├── services/             # Business logic layer
│   ├── docker/           # DockerService (dockerode), DockerComposeService (execa), container/network mgmt
│   ├── sites/            # SiteService, N8NSiteService, N8NSiteConfigurator - instance lifecycle
│   ├── proxy/            # NginxProxyService, NginxService, DomainService - reverse proxy config
│   ├── database/         # PostgreSQLService, RedisService - shared DB container management
│   ├── backup/           # BackupService, BackupScheduler (node-cron)
│   ├── ssl/              # SSLService - self-signed and Let's Encrypt certificates
│   └── filesystem/       # FileSystemService - local and SFTP (ssh2-sftp-client) file ops
├── models/               # Site.js (instance model), AuthKey.js, Settings.js (env + INI config)
├── N8NManager.js         # Central orchestrator: initializes PostgreSQL, Redis, Docker network, Nginx
└── resources/            # Nginx config templates (Handlebars), backup storage
```

**Key patterns:**
- Controllers call Services, Services manage Docker/filesystem/database
- Auth is API-key based: `x-api-key` header or Bearer token, validated by `middleware/auth.js`
- Docker operations use `dockerode` for API calls and `execa` for docker-compose CLI
- Supports remote host operations via SFTP (configured with WORK_HOST env vars)
- Nginx configs are generated from Handlebars templates in `resources/`

### Frontend (React 18 + Vite, port 5173)

```
app/frontend/src/
├── App.jsx               # Router setup, AuthProvider + ThemeProvider
├── pages/                # Dashboard, Instancias, Databases, EditorSQL, Templates, Backups, Nginx, Login
├── components/           # Layout, Sidebar, dialogs (CreateInstance, EditInstance), SystemServices
│   └── ui/               # ShadCN UI components (Radix UI primitives)
├── contexts/             # AuthContext (API key auth), ThemeContext (dark mode)
├── lib/api.js            # Axios instance with x-api-key interceptor, 401 redirect
└── hooks/                # use-toast
```

**Key patterns:**
- ShadCN UI component library with TailwindCSS
- Vite proxies `/api` to `http://localhost:3000` in dev
- Auth state stored in `localStorage.auth_key`

### Docker Container Naming

- `n8nmanager-database` (PostgreSQL 16 Alpine)
- `n8nmanager-redis` (Redis 7 Alpine)
- `n8nmanager-proxy` (Nginx, ports 80/443)
- `{siteName}_instance` and `{siteName}_worker` (N8N instances)
- All on network: `n8nmanager-network`

## Key Environment Variables (backend .env)

- `AUTH_KEY` - API authentication key
- `POSTGRES_USER`, `POSTGRES_PASSWORD` - shared PostgreSQL credentials
- `REDIS_PASSWORD` - shared Redis password
- `WORK_PATH`, `SITES_PATH` - filesystem paths for managed data
- `FRONTEND_URL`, `FRONTEND_SERVER_URL` - CORS origins
- `WORK_HOST`, `HOST_PORT`, `HOST_USER`, `HOST_PASS` - remote host (SFTP) config

---
> Source: [Xfirepc/n8n-octopus](https://github.com/Xfirepc/n8n-octopus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
