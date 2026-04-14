## hub-nsane-digital

> This project is deployed via Dokploy with auto-deploy on push to `main`.

@AGENTS.md

## Dokploy Deployment

This project is deployed via Dokploy with auto-deploy on push to `main`.

### Identifiers

> **First session setup:** Use `mcp__dokploy-mcp__project-all` to find this project by name,
> then fill in the IDs below. Update this file so future sessions skip the lookup.

| Resource    | ID                           |
|-------------|------------------------------|
| Project     | `<lookup via project-all>`   |
| Environment | `<lookup via project-one>`   |
| Application | `<lookup via project-one>`   |
| App Name    | `<lookup via application-one>` |

### Deployment Workflow

1. Push to `main` triggers auto-deploy via GitHub webhook
2. Dokploy builds using the `Dockerfile` (4-stage multi-stage: base → deps → builder → runner)
3. On container start, `scripts/start.js` purges Cloudflare cache then starts the Next.js server

### Common Operations (via Dokploy MCP)

- **Check status**: `application-one` with applicationId
- **Manual deploy**: `application-deploy` with applicationId (uses latest commit on `main`)
- **Redeploy**: `application-redeploy` with applicationId (rebuilds from same commit)
- **Start/Stop**: `application-start` / `application-stop`
- **View monitoring**: `application-readAppMonitoring` with appName
- **Update env vars**: `application-saveEnvironment` with applicationId
- **Cancel deployment**: `application-cancelDeployment` with applicationId
- **View domains**: `domain-byApplicationId` with applicationId

### Environment Variables

Managed in Dokploy, not committed. See `.env.example` for required vars.

### Troubleshooting

- If deploy fails, check deployment logs via `application-one` (deployments array has status + logPath)
- If app is unresponsive, check monitoring via `application-readAppMonitoring`
- To force clean build, use `application-redeploy` or set `cleanCache: true` via `application-update`
- If deploy is stuck, use `application-cancelDeployment` then `application-cleanQueues`

## Playwright MCP

Use Playwright MCP tools (`mcp__playwright`) for debugging, visual testing, and verifying the app in a browser.
Save all screenshots and generated files to `.playwright-mcp/` (gitignored).

When taking screenshots, use a path like `.playwright-mcp/screenshot-<description>.png`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neupauer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/neupauer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
