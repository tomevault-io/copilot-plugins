## slate-app-test

> This is a Slyte app. Prefer Slyte CLI and framework conventions over generic Node/React patterns.

# Slyte project: slyteApp

## For AI assistants

This is a Slyte app. Prefer Slyte CLI and framework conventions over generic Node/React patterns.

### Enable Slyte MCP (recommended)

Ask the user to enable the Slyte MCP server if they want CLI-aware help (generate, destroy, docs, project structure).
Without MCP, you can still help by reading this file and the app sources, then suggesting exact `slyte` commands for the user to run.

## Project layout

- `app.js` — app entry
- `components/` — UI components (javascript, templates, styles)
- `router/` — router, maps, and route handlers
- `data-store/` — db, schemas, connectors, serializers
- `build/build.js` — build/serve options
- `slyte.config.js` — app config

## Common CLI commands

```bash
npm start
npm run build:prod
npm run slyte -- generate component <name>
npm run slyte -- generate route <name>
npm run slyte -- generate map <name>
npm run slyte -- destroy <module> <name>
npm run slyte -- install <package>
```

Use `slyte generate --help` / `slyte destroy --help` for the full module list.

## Conventions

- Prefer CLI generators over hand-rolling boilerplate files.
- Keep component / route / schema names aligned with Slyte naming rules (kebab-case entity names).
- Do not invent APIs that are not in Slyte docs or this project's existing code.
- Avoid committing build output (`dist/`) or local Vite helper folders under `build/`.

## Docs

Consult Slyte docs for router maps, schemas, component registries, and data-store wiring before changing architecture.

---
> Source: [ragulpandi/slate-app-test](https://github.com/ragulpandi/slate-app-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
