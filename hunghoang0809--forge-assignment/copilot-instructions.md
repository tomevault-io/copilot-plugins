## forge-assignment

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains three independent Atlassian Forge apps deployed on the Forge Cloud platform. They are **not** a monorepo — each app has its own manifest, dependencies, and deployment lifecycle. All Forge CLI commands (except `create`, `version`, `login`) must be run from the specific app's root directory.

| App | Directory | Module | Product |
|-----|-----------|--------|---------|
| Custom Space Page | `Custom-Space-Page/` | `confluence:spacePage` | Confluence |
| Issue Context | `Issue-Context-App/` | `jira:issueContext` | Jira |
| Issue Panel | `issue-panel-custom-app/` | `jira:issuePanel` | Jira |

## Common Commands

All commands must be run from the **app's root directory** (e.g., `cd Custom-Space-Page` first).

```bash
# Lint (npm)
npm run lint                  # ESLint on src/**/*

# Lint (Forge CLI)
forge lint                    # Validates manifest.yml and code

# Deploy
forge deploy --non-interactive -e development

# Install (first time)
forge install --non-interactive --site <site-url> --product <product>

# Upgrade (after scope/permission changes)
forge install --non-interactive --upgrade --site <site-url> --product <product> --environment development

# Local development
forge tunnel                  # Hot-reloads code changes; restart tunnel after manifest changes

# Debugging
forge logs                    # View app logs (default: last 15 min)
forge logs -n 50              # Last 50 lines
forge logs -e production      # Production environment logs
forge logs --since 2d         # Logs from last 2 days
```

## Architecture

Each app follows an identical pattern:

```
src/
  frontend/index.jsx     # React UI Kit entry point (renders via ForgeReconciler)
  index.js               # Backend entry (re-exports from resolvers)
  resolvers/index.js     # Resolver functions called from frontend via invoke()
```

**Data flow:** Frontend (`invoke('resolverName', payload)`) → Resolver (`@forge/resolver`) → Product REST API (`@forge/api`) → Return data → Frontend renders.

- `manifest.yml` wires frontend resources to modules and maps resolvers to function handlers.
- Resolvers use `api.asApp().requestJira()` or `api.asApp().requestConfluence()` with `route` tagged templates for API calls.
- Frontend uses `invoke()` from `@forge/bridge` to call resolvers.

## Forge Development Rules

- **UI Kit only** — use components from `@forge/react` exclusively. No standard React DOM elements (`<div>`, `<strong>`, etc.). No `@forge/ui` (deprecated).
- **No "Table" component** — use `DynamicTable` instead.
- **Prefer `requestJira`/`requestConfluence` on the frontend** over backend resolvers when possible.
- **Prefer `.asUser()`** over `.asApp()` in resolvers when making product API requests, unless app-level permissions are needed.
- **Minimize scopes** — only add permissions to `manifest.yml` when required by the API.
- **After changing `manifest.yml` scopes or egress**: redeploy AND reinstall the app.
- **When tunneling**: code-only changes hot-reload; manifest changes require redeploy + tunnel restart.
- **Never use `--no-verify`** flag with `forge deploy` unless explicitly requested.
- **Create new apps** with `forge create -t <template> <name>` — never modify an existing app's directory for a new project.

## Dependencies

| Package | Role |
|---------|------|
| `@forge/react` | UI Kit component library (frontend) |
| `@forge/bridge` | Client-side invoke API (frontend) |
| `@forge/resolver` | Backend resolver framework |
| `@forge/api` | Server-side authenticated product API calls |

Note: `@forge/api` versions differ across apps (v2.0.0 in Custom-Space-Page and issue-panel-custom-app, v7.1.4 in Issue-Context-App).

---
> Source: [hunghoang0809/Forge-Assignment](https://github.com/hunghoang0809/Forge-Assignment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
