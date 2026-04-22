## conductor

> Orchestration hub for DuoCircle's AI agent workforce and Dockerized tools, running on Proxmox.

# Conductor

Orchestration hub for DuoCircle's AI agent workforce and Dockerized tools, running on Proxmox.

## Machines

| Machine | LAN IP | Tailscale | Name |
|---|---|---|---|
| Proxmox Host | 192.168.2.172:8006 | — | conductor (hypervisor) |
| Main VM | 192.168.2.177 | 100.112.37.68 | conductor-main |

## Services (all on conductor-main)

| Service | Port | URL (Tailscale) | Purpose |
|---|---|---|---|
| Nginx Proxy Manager | 81 | http://conductor-main:81 | Reverse proxy admin |
| Infisical | 8443 | https://conductor-main:8443 | Secrets management (HTTPS) |
| Mem0 | 8070 | http://conductor-main:8070 | Shared agent memory API |

## Quick Commands

```bash
# SSH to VM
ssh brad@192.168.2.177
ssh brad@conductor-main  # via Tailscale

# Start all services
cd ~/conductor && docker compose up -d

# Start Oliver (marketing agent)
docker compose -f agents/oliver/docker-compose.yml up -d

# View logs
docker compose logs -f mem0
docker compose logs -f infisical

# Mem0 health check
curl http://conductor-main:8070/health
```

## Directory Structure

- `agents/` — Agent definitions (each has own docker-compose + config)
- `services/` — Service-specific Dockerfiles and configs
- `scripts/` — Setup and migration scripts
- `infrastructure/` — Proxmox and Tailscale notes

## Secrets

All secrets in `.env` on the VM (gitignored). `.env.example` has the template.
Never commit API keys, tokens, or passwords.

## Mem0 Shared Memory API

All agents connect to Mem0 for shared organizational knowledge.

```bash
# Health check
curl http://conductor-main:8070/health

# Add a memory
curl -X POST http://conductor-main:8070/v1/memories \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "DuoCircle sells DMARC Report for $3900/domain"}], "agent_id": "oliver-marketing"}'

# Search memories
curl -X POST http://conductor-main:8070/v1/memories/search \
  -H "Content-Type: application/json" \
  -d '{"query": "DMARC pricing", "agent_id": "oliver-marketing"}'
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bslavin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
