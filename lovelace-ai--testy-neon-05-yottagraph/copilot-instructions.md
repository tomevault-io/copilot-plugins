## testy-neon-05-yottagraph

> Project structure, navigation patterns, data architecture, server routes, agents, MCP servers. Read when adding pages, navigation, or server-side functionality.


# Aether Architecture

Aether is an app framework built on Nuxt 3 + Vue 3 + Vuetify 3 + TypeScript. It follows standard Nuxt conventions -- pages in `pages/`, components in `components/`, composables in `composables/`, server routes in `server/api/`.

**Tech stack**: Nuxt 3 (SPA), Vue 3 Composition API (`<script setup>`), Vuetify 3, TypeScript (required), Auth0 (automatic).

**Data source**: This app runs on the Lovelace platform. The Query Server (Elemental API) is the primary data source -- use it for entities, news, filings, sentiment, relationships, and events. See the `api` rule. Do not call external APIs for data the platform provides.

## Project Structure

```
pages/              # Routes (file-based routing)
components/         # Reusable Vue components
composables/        # Shared logic (useXxx.ts)
server/api/         # Nitro server routes (deploy with app)
assets/             # CSS, fonts, static assets
utils/              # Pure utility functions
agents/             # ADK agents (Python, deploy to Vertex AI)
mcp-servers/        # MCP servers (Python, deploy to Cloud Run)
```

## Data Architecture

| Store | Purpose | When to use |
|---|---|---|
| Query Server (Elemental API) | Lovelace Knowledge Graph | Entities, news, relationships, events, sentiment, filings |
| KV (Upstash Redis) | User preferences, lightweight state | Settings, watchlists, UI state that should persist |
| Neon Postgres | App-specific relational data | Custom tables, complex queries (if connected — check `DATABASE_URL` in `.env`) |

Use `useElementalClient()` for Query Server data (see `api` rule).
Use `Pref<T>` or `usePrefsStore()` for KV (see `pref` rule).
Check `.env` for `DATABASE_URL` before using Neon Postgres (see `server` rule).

## Adding Pages

Create `.vue` files in `pages/`. Nuxt generates routes automatically:

```
pages/index.vue         → /
pages/dashboard.vue     → /dashboard
pages/settings.vue      → /settings
pages/reports/index.vue → /reports
```

## App Shell

`app.vue` provides `AppHeader` (branding, user menu, settings) and renders `<NuxtPage />`. There is no prescribed layout beyond the header -- design whatever UX fits the problem.

## Adding Navigation

Only add navigation if the app genuinely needs multiple views. Choose the pattern that fits the UX:

**Sidebar** -- add to `app.vue` for persistent section navigation:

```vue
<v-navigation-drawer permanent app>
  <v-list nav density="compact">
    <v-list-item title="Dashboard" prepend-icon="mdi-view-dashboard" to="/" />
    <v-list-item title="Reports" prepend-icon="mdi-chart-bar" to="/reports" />
  </v-list>
</v-navigation-drawer>
```

**Top tabs** -- add to `app.vue` or a layout component:

```vue
<v-tabs>
  <v-tab to="/">Dashboard</v-tab>
  <v-tab to="/reports">Reports</v-tab>
</v-tabs>
```

**In-page tabs** -- for subsections within a single page:

```vue
<v-tabs v-model="tab">
  <v-tab value="overview">Overview</v-tab>
  <v-tab value="details">Details</v-tab>
</v-tabs>
<v-window v-model="tab">
  <v-window-item value="overview">...</v-window-item>
  <v-window-item value="details">...</v-window-item>
</v-window>
```

**No nav** -- single-page apps can put everything on `pages/index.vue`.

## Naming Conventions

- **Pages**: `kebab-case.vue`
- **Components**: `PascalCase.vue`
- **Composables**: `useCamelCase.ts`
- **Server routes**: `kebab-case.<method>.ts`

## New Page Checklist

- [ ] Created a design doc in `design/` (copy `design/feature_template.md`)
- [ ] Page in `pages/` with `<script setup lang="ts">`
- [ ] Uses Vuetify components and the project's dark theme
- [ ] Updated `DESIGN.md` with current status

## Server Routes

Nitro server routes live in `server/api/` and deploy with the app to Vercel. They're auto-registered by Nuxt:

```
server/api/my-data/fetch.get.ts   →   GET /api/my-data/fetch
server/api/my-data/submit.post.ts →  POST /api/my-data/submit
```

Call them from client code with `$fetch('/api/my-data/fetch')`. See the `server` cursor rule for patterns. Use server routes when you need to proxy external APIs (avoid CORS) or keep secrets off the client.

## Beyond the UI: Agents, MCP Servers, and Server Routes

Aether apps are more than just a Nuxt SPA. The project contains three additional directories that deploy independently:

### `agents/` -- ADK Agents (Python)

Each subdirectory is a self-contained Python agent that deploys to Vertex AI Agent Engine. See the `agents` cursor rule for development patterns. Agents are deployed via the Broadchurch Portal UI or the `/deploy_agent` Cursor command, both of which trigger `deploy-agent.yml`. Once deployed, agents are reachable through the Portal Gateway (`NUXT_PUBLIC_GATEWAY_URL`). Use the `useAgentChat` composable to build a chat UI that talks to them.

### `mcp-servers/` -- MCP Servers (Python)

Each subdirectory is a FastMCP server that deploys to Cloud Run. See the `mcp-servers` cursor rule. Deployed via Portal UI or `/deploy_mcp`, triggering `deploy-mcp.yml`. Agents can connect to MCP servers as tool providers.

### `broadchurch.yaml`

Tenant-specific configuration generated during provisioning. Contains GCP project, org ID, service account, gateway URL, and query server URL. Read by deploy commands and GitHub Actions workflows. Don't edit manually.

## Available Composables for Platform Features

The template ships composables for interacting with the Lovelace platform.
Use these to build whatever UI fits the app:

- **`useAgentChat()`** -- Send messages to deployed ADK agents through the
  Portal Gateway. Handles streaming, session management, and response
  parsing. See the `agents` cursor rule for gateway endpoints and response
  formats.
- **`useTenantConfig()`** -- Fetch the tenant's runtime config (deployed
  agents, feature flags, MCP servers) from the Portal Gateway.
- **`useElementalClient()`** -- Query Server client for entities, news,
  filings, etc. See the `api` rule.
- **`usePrefsStore()` / `Pref<T>`** -- KV-backed user preferences. See
  the `pref` rule.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovelace-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
