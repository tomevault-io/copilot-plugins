## homelab-openclaw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-host Ansible project deploying OpenClaw on `openclaw.local.iamrobertyoung.co.uk`. Secrets are stored in AWS SSM Parameter Store. Priority is security as personal information may be stored.

## Key Commands

All commands require AWS credentials via aws-vault for SSM parameter lookups.

```bash
# Install external dependencies (roles)
aws-vault exec iamrobertyoung:home-assistant-production:p -- ansible-galaxy install -r requirements.yml -p .roles

# Test connectivity
aws-vault exec iamrobertyoung:home-assistant-production:p -- ansible openclaw -m ping

# Run full playbook
aws-vault exec iamrobertyoung:home-assistant-production:p -- ansible-playbook playbooks/site.yml

# Run specific role only
aws-vault exec iamrobertyoung:home-assistant-production:p -- ansible-playbook playbooks/site.yml --tags openclaw

# Lint
yamllint .
ansible-lint

# Verify service health after deploy
curl -fsS https://openclaw.local.iamrobertyoung.co.uk/healthz
curl -fsS https://openclaw.local.iamrobertyoung.co.uk/readyz
```

## Architecture

```
Internet → UFW (443 allowed, 18789 blocked) → Traefik (Docker, TLS via Step CA)
         → host.docker.internal:18789 → OpenClaw gateway (host systemd service, as `openclaw` user)
                                       → Docker agent sandboxes
```

The OpenClaw gateway runs **on the host** (not in Docker). Docker is used only for the Traefik reverse proxy and agent sandboxes.

### Traffic Flow
1. External HTTPS traffic hits port 443
2. UFW allows 443, blocks direct access to 18789
3. Traefik (Docker container) terminates TLS using Step CA certs
4. Traefik proxies to `host.docker.internal:18789` via `extra_hosts: host-gateway`
5. OpenClaw gateway (systemd service, `openclaw` user) handles the request

### Deployment Stack
The main playbook (`playbooks/site.yml`) applies these roles in order:
1. `configure-system` - Base system configuration
2. `shell` - Shell setup for users (noxious, root)
3. `docker` - Docker installation
4. `step-ca-client` - TLS certificates from Step CA (post-renew: copy certs, HUP traefik, restart openclaw)
5. `telegraf` - Metrics collection to InfluxDB at `influxdb.local.iamrobertyoung.co.uk`
6. `syslog` - Syslog configuration
7. `wazuh-agent` - Security monitoring agent
8. `openclaw` - OpenClaw deployment (custom role)

### OpenClaw Role Structure
The `roles/openclaw` role deploys OpenClaw and Traefik:
- `tasks/main.yml` - Orchestration: setup → ufw → nodejs → openclaw → traefik
- `tasks/setup.yml` - User/dir creation, TLS cert copy, config template
- `tasks/ufw.yml` - Firewall rules (allow 22, 443; deny 18789; default deny)
- `tasks/nodejs.yml` - Node.js 22.x + pnpm installation
- `tasks/openclaw.yml` - pnpm install + systemd service
- `tasks/traefik.yml` - Traefik config files + systemd service
- `defaults/main.yml` - Default variables
- `handlers/main.yml` - Handler chains: stop → reload systemd → start

### Key Directories on Target Host
- `/etc/openclaw` - Configuration files (mode 0750)
- `/var/lib/openclaw` - Application state (mode 0750)
- `/var/log/openclaw` - Log files (mode 0750)
- `/etc/traefik` - Traefik config and docker-compose.yml
- `/etc/traefik/certs` - TLS certificates (copied from /etc/ssl)
- `/etc/traefik/dynamic` - Traefik dynamic configuration

### Secrets Management
Secrets are stored in AWS SSM Parameter Store (region: eu-west-2).
Ansible retrieves them via `lookup('aws_ssm', '/path/to/secret', region='eu-west-2')`.

| SSM Parameter | Purpose |
|---|---|
| `/home-server/step-ca-ansible-password` | Step CA provisioner password |
| `/home-server/influxdb-token` | InfluxDB metrics token |
| `/openclaw/gateway-token` | OpenClaw gateway auth token |

### Security Hardening
| Layer | Mechanism |
|---|---|
| Network | UFW default deny; allow 22, 443; explicitly deny 18789 |
| TLS | Step CA certs, TLS 1.2 min, SNI strict, renewed every 7 days |
| Process | systemd: `NoNewPrivileges`, `ProtectSystem=strict`, `PrivateTmp`, `PrivateDevices` |
| User | Dedicated `openclaw` user with `nologin` shell, no home directory |
| Secrets | AWS SSM Parameter Store; gitleaks pre-commit hook; config mode 0640 |
| Traefik dashboard | Disabled (security) |

### External Dependencies
Dependencies in `requirements.yml` are installed to `.roles/` (gitignored). External roles use SSH git URLs requiring SSH keys.

### Available Tags
Run specific parts with `--tags`: `configure-system`, `shell`, `docker`, `step-ca-client`, `telegraf`, `syslog`, `wazuh-agent`, `openclaw`, `setup`, `ufw`, `nodejs`, `traefik`

### Tool Versions (mise.toml)
- ansible 13.1.0
- pipx 1.8.0
- uv 0.9.18

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RobertYoung) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
