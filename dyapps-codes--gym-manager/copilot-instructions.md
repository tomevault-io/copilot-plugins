## gym-manager

> Instructions for building apps with DYPAI MCP


# DYPAI — AI Agent Instructions

You are building full-stack applications on the DYPAI platform using MCP tools.

## Getting Started
1. Call `list_projects()` to find your project_id
2. Use `get_app_tables` and `search_endpoints` to understand existing state
3. Before implementing any feature, call `search_docs` with the topic to get correct SDK patterns
4. Build backend first (tables + endpoints), then frontend

## Build Backend
- Create tables with `execute_sql` (check `get_app_tables` first)
- Create endpoints with `create_endpoint` using workflow_code
- Test with `test_workflow` after creating/updating
- Use `search_templates` for ready-made workflow patterns
- Use `search_nodes` to discover available node types

## Build Frontend
- SDK client: `import { dypai } from './lib/dypai'`
- API calls: `dypai.api.get(name)`, `dypai.api.post(name, body)`
- Auth: `dypai.auth.signInWithPassword()`, `signUp()`, `signOut()`
- Files: `dypai.api.upload(name, file)`
- Every method returns `{ data, error }`
- Do NOT use fetch() directly — always use the SDK

## Deploy
- Use `deploy_frontend` with the project's source directory
- Site goes live at `https://{slug}.dypai.app` within ~1 minute
- Check progress with `get_build_status`

## Rules
- Auth modes: jwt (default), api_key (server-to-server), public (read-only)
- NEVER use public for endpoints that write data
- Do NOT create auth endpoints — auth is built-in via SDK
- Always test endpoints after creating them

---
> Source: [dyapps-codes/gym-manager](https://github.com/dyapps-codes/gym-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
