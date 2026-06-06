## oex-downloader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Self-hosted torrent proxy stack that routes all qBittorrent traffic through xray-core's SOCKS5h listener via tun2socks sidecar. Three-container Docker Compose deployment:
- **xray**: VLESS/Reality proxy with local SOCKS5 inbound (default port 1080)
- **tun2socks**: TUN interface inside container, bridges container traffic to xray SOCKS5h
- **qbittorrent**: Torrent client sharing tun2socks's network namespace — forced through proxy with kill-switch (if xray/tun2socks down, qBittorrent has no network path)

WebUI (port 8080) binds to loopback only; accessed via SSH local-forward.

## Configuration flow

1. **config/stack-settings.json** — user settings (WebUI port, PUID/PGID, SOCKS5 auth, VPS IP for leak detection, TUN interface name)
2. **scripts/generate-env.sh** — reads stack-settings.json, writes .env with Docker Compose variables
3. **config/xray-config.json** — xray routing (sourced from xray-config.json.example; requires PLACEHOLDER fills: VLESS server IP, UUID, Reality pubkey/shortId/SNI)
4. **config/qbittorrent.conf** — qBittorrent template (seeded once to data/qbittorrent/config/qBittorrent/qBittorrent.conf)
5. **scripts/verify.sh** — comprehensive health check: container states, egress IPs, UDP DNS, DoH reachability, kill-switch

## Common commands

```bash
# First-time setup (creates xray-config.json, qBittorrent.conf, .env from templates)
bash scripts/bootstrap.sh

# Edit your VLESS credentials and Reality settings
$EDITOR config/xray-config.json

# Edit runtime settings (port, PUID/PGID, VPS IP for leak detection, etc.)
$EDITOR config/stack-settings.json
bash scripts/bootstrap.sh    # regenerates .env

# Start the stack
docker compose up -d

# Check status and logs
docker compose ps
docker compose logs xray --tail=20
docker compose logs tun2socks --tail=20
docker compose logs qbittorrent --tail=20

# Comprehensive verification (health, egress IP, DNS, DoH, kill-switch readiness)
bash scripts/verify.sh

# Stop / restart
docker compose stop [service]
docker compose restart [service]
docker compose down

# Update images
docker compose pull
docker compose up -d --remove-orphans
```

## Key design notes

- **No host VPN/routing changes** — only container netns affected; SSH untouched.
- **TUN/TAP device required** — `/dev/net/tun` must exist on host; most KVM VPS have it, some LXC don't.
- **NET_ADMIN capability needed** — test with `docker run --rm --cap-add NET_ADMIN alpine ip link`.
- **Kill switch built-in** — qBittorrent shares tun2socks's netns with no alternative network path; if xray/tun2socks fail, container is offline.
- **SOCKS5h required** — tun2socks needs SOCKS5h (hostname resolution) support; plain SOCKS5 will leak on DNS.
- **UDP support** — xray inbound must have `"udp": true` for DHT/DNS/µTP to work; if disabled, UDP traffic silently fails (logs show `UDP ASSOCIATE: command not supported`).
- **DNS over HTTPS (DoH) mandatory** — xray should route DNS to DoH endpoints (e.g., `https://1.1.1.1/dns-query`), not plaintext UDP :53, to avoid provider/ISP DNS sniffing.

## Configuration placeholders

**config/xray-config.json outbounds** — VLESS upstream must be filled:
- `outbounds[0].settings.vnext[0].address` — VLESS server hostname/IP
- `outbounds[0].settings.vnext[0].port` — VLESS port (usually 443)
- `outbounds[0].settings.vnext[0].users[0].id` — Client UUID
- `outbounds[0].settings.vnext[0].users[0].flow` — XTLS flow (e.g., `xtls-rprx-vision`); omit if not Reality
- `streamSettings.realitySettings.serverName` — SNI for Reality
- `streamSettings.realitySettings.publicKey` — Reality server public key
- `streamSettings.realitySettings.shortId` — Reality short ID

If not using VLESS+Reality, replace the entire `vless-out` outbound with your protocol.

## Troubleshooting quick-ref

| Symptom | Check |
|---|---|
| tun2socks unhealthy | `docker compose logs xray` — xray must be reachable; inbound needs `"listen": "0.0.0.0"`, `"udp": true` |
| Egress shows VPS IP | `docker compose exec tun2socks ip route` — should have `default dev tun0` |
| DNS fails | xray inbound must have `"udp": true`; logs show `UDP ASSOCIATE` error if not |
| qBittorrent shows 0 DHT nodes | `data/qbittorrent/config/qBittorrent/qBittorrent.conf`: set `Session\Interface=tun0` |
| WebUI unreachable | SSH tunnel running? `ssh -L 8080:127.0.0.1:8080 user@vps` |
| `/dev/net/tun` missing | Enable TUN in VPS panel or switch to TCP-only proxy |
| `NET_ADMIN` denied | LXC container without capability — need KVM-based VPS |
| Healthcheck shows leak | Verify `vps_public_ip` in stack-settings.json; if correct, investigate routes |

## Scripts

- **bootstrap.sh** — One-time init: copies xray-config.json.example → xray-config.json (if missing), seeds qBittorrent.conf, mkdir data dirs, calls generate-env.sh.
- **generate-env.sh** — Reads config/stack-settings.json (JSON), writes .env with Docker Compose vars; requires jq or python3.
- **verify.sh** — Multi-step health check: container states, egress IP, DNS/DoH reachability, leak detection, kill-switch readiness (skips step 8 manual magnet test).
- **healthcheck-proxy.sh** — Container healthcheck script (runs inside tun2socks); checks TUN is up and SOCKS5 reachable.
- **sub-manager.py** — Subscription manager (dev utility); parses base64-encoded proxy lists (v2ray, Clash formats), tests connectivity via downloaded xray binary.

## Development environment

**devbox.json** defines:
- `ansible@latest` — for deploy scripts
- `python@3.14.3` — for subscription manager and utilities

Packages available in nix shells via devbox; can run `devbox shell` to enter isolated environment.

## Deployment

**deploy/** directory contains Ansible playbook for remote deployment:
- **ansible.cfg** — ansible config
- **deploy.yml** — playbook to orchestrate bootstrap, config, docker compose up
- **inventory.ini** — VPS target hosts
- **README.md** — deployment guide

## File structure notes

- **config/** — All user-editable config; stack-settings.json is the main knob
- **data/** — Persistent volumes: xray state, qBittorrent config, torrent downloads
- **.env** — Generated by generate-env.sh from stack-settings.json; controls Docker Compose substitutions; not committed
- **docker-compose.yml** — Service definitions; uses env vars from .env for port/auth/paths
- **scripts/** — Operational scripts (bootstrap, verify, healthcheck)

---
> Source: [1-1is0/oex-downloader](https://github.com/1-1is0/oex-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
