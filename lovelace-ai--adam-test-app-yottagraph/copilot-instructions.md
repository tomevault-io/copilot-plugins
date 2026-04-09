## adam-test-app-yottagraph

> **Stack:** Nuxt 3 (SPA), Vue 3 Composition API (`<script setup>`), Vuetify 3, TypeScript (required), Auth0.


# Aether

**Stack:** Nuxt 3 (SPA), Vue 3 Composition API (`<script setup>`), Vuetify 3, TypeScript (required), Auth0.

**Structure:** `pages/` (file-based routing), `components/`, `composables/`, `server/api/`, `agents/` (Python ADK), `mcp-servers/` (Python FastMCP).

**Data:** The Query Server (Elemental API) is the primary data source -- entities, news, filings, sentiment, relationships, events. Use `useElementalClient()` from `@yottagraph-app/elemental-api/client`. Do NOT call external APIs for data the platform provides. Discovery-first: use `getSchema()` to discover entity types and properties at runtime. Skill docs in `skills/elemental-api/` (run `npm install` if empty). Lovelace MCP servers may also be configured (see `api` rule) but are optional — check your tool list before assuming they're available.

**Storage:** KV (Upstash Redis) for preferences and lightweight state via `Pref<T>` from `usePrefsStore()`. Supabase for relational data if connected (check `.env`).

**Source of truth:** `DESIGN.md` -- read before starting work, update when changing features. The starter UI is placeholder -- replace freely. Feature docs in `design/` for implementation planning.

**Git:** Commit meaningful units of work. Run `npm run format` before commit. Message format: `[Agent commit] {summary}`.

**First action for a new project:** Run `/build_my_app`.

**Task-specific rules:** `architecture` (project structure, navigation, server routes, agents, MCP), `api` (Elemental API client, schema discovery, gotchas, optional MCP servers), `design` (DESIGN.md workflow, feature docs), `ui` (page templates, layout patterns), `cookbook` (copy-paste UI patterns), `pref` (KV preferences), `branding` (colors, fonts), `server` (Nitro routes, Supabase), `something-broke` (error recovery, build failures).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovelace-AI)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Lovelace-AI)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
