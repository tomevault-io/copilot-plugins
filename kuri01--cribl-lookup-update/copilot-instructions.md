## cribl-lookup-update

> This repository contains a local Cribl environment plus two helper APIs:

# AGENTS.md

## Purpose
This repository contains a local Cribl environment plus two helper APIs:
- `mock-jira-insight` (source of CMDB-like JSON data)
- `jira-cmdb-api` (fetches data from `mock-jira-insight`, converts to CSV, updates/deploys Cribl lookup)

Use this file as the default guidance for AI/code agents working in this repo.

## Architecture Rules
- Keep `mock-jira-insight` small and simple.
  - It should only expose mock Jira Insight data.
  - Primary endpoint: `GET /insight/objects`.
  - Data source: `mock-jira-insight/data.json`.
- Keep Cribl integration logic in `jira-cmdb-api`.
  - It owns: auth, lookup upload/replace/create, and deploy flow.
  - Primary endpoint: `POST /cribl/lookups/update`.
- `jira-cmdb-api` must fetch input data from `mock-jira-insight` over HTTP (not from local file).

## Naming and Service Conventions
- Compose service names:
  - `mock-jira-insight`
  - `jira-cmdb-api`
- Internal URLs:
  - `http://mock-jira-insight:3000/insight/objects`
  - `http://jira-cmdb-api:3000/insight/objects`
  - `http://jira-cmdb-api:3000/cribl/lookups/update`
- Host URLs:
  - `http://localhost:13001/insight/objects` (mock-jira-insight)
  - `http://localhost:13000/insight/objects` (jira-cmdb-api passthrough)
  - `http://localhost:13000/cribl/lookups/update`

## Environment and Secrets
- Main runtime config is in root `.env` (see `.env.example`).
- Required for Cribl auth:
  - `CRIBL_API_USERNAME`
  - `CRIBL_API_PASSWORD`
- Optional fallback:
  - `CRIBL_API_TOKEN`
- Never commit real credentials/tokens.

## Build/Run Expectations
- Docker-first workflow is preferred.
- Host Node.js is optional if using Docker Compose.
- Standard run command:
  - `docker compose up -d --build`

## Cribl Lookup Behavior (Must Preserve)
- Update endpoint flow:
  1. Upload CSV to Cribl lookup temp file.
  2. Try replace existing lookup.
  3. If lookup missing, create it.
  4. Deploy lookup changes to worker group.
- Default behavior should deploy changes (`deploy: true` unless explicitly disabled).
- Deploy payload should preserve other existing lookups in the group payload (do not accidentally drop them).

## Testing and Validation
- Preferred checks after code changes:
  - `npm run build` (root turbo build)
  - `npm run test:e2e -- --watchman=false` in `jira-cmdb-api`
  - `docker compose config`

## Safe Change Guidelines
- Avoid renaming services, env var keys, or endpoints unless explicitly requested.
- Keep docs (`README.md`) updated when behavior/config changes.
- Keep changes minimal and focused; avoid unrelated refactors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kuri01)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kuri01)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
