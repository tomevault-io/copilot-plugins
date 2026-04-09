## sigma

> Sigma is a lightweight VPS fleet management platform for a VPN infrastructure project. The operator manages VPS instances across **dozens of small cloud providers** in many countries. VPS turnover is high — instances typically last ~1 month before being replaced.

# Σ Sigma — Project Context

## What is this?

Sigma is a lightweight VPS fleet management platform for a VPN infrastructure project. The operator manages VPS instances across **dozens of small cloud providers** in many countries. VPS turnover is high — instances typically last ~1 month before being replaced.

## Repo Structure

```
sigma/
├── sigma-api/          # Rust backend (Axum + SQLx + PostgreSQL)
├── sigma-web/          # React frontend (Vite + TypeScript + Tailwind CSS)
├── sigma-cli/          # Rust CLI client (clap + reqwest)
├── sigma-probe/        # IP reachability probe (deployed on China nodes)
├── sigma-agent/        # VPS system agent (auto-register + heartbeat + port scan + metrics)
├── grafana/            # Grafana dashboard JSON definitions
├── k8s/                # Kubernetes deployment configs
├── docker-compose.yml  # Dev/staging orchestration (db + api + web + probe + agent)
├── Makefile            # Common commands
└── DEPLOYMENT.md       # Deployment guide
```

## Key Design Decisions

- **Rust + Axum + SQLx + PostgreSQL** — single binary deployment, low resource usage
- **React + Vite + TypeScript + Tailwind CSS** — professional web UI for non-tech operators
- **PostgreSQL** — leverages JSONB, TEXT[], GIN indexes for IP and tag queries
- **No ORM magic** — raw SQL via sqlx::query_as for full control and transparency
- **Runtime migrations** — using `sqlx::migrate::Migrator::new()` instead of compile-time `migrate!()` macro
- **JWT + API key dual auth** — email/password login with JWT (Bearer token), plus multi-API-key management (`X-Api-Key`) with per-key roles. Four roles: admin, operator, agent, readonly
- **Prometheus file_sd integration** — core integration point with Thanos/Prometheus/Grafana stack
- **Docker Compose (dev)** → **Kubernetes (production)**

## Data Model

- **Provider** — cloud platform vendor (e.g. any VPS hosting company)
- **VPS** — individual server instance with lifecycle: provisioning → active → retiring → retired
- **ip_checks** — table for tracking IP reachability from China (API done, probe: `sigma-probe/`)
- **envoy_routes** — Envoy forwarding routes with `source` column (`dynamic` = managed via API/UI, `static` = synced from envoy.yaml)
- **cloud_accounts** — cloud provider accounts (AWS, Alibaba Cloud, DigitalOcean, Linode, Volcengine) with `provider_type` and JSONB `config`; manual sync pulls all instances and upserts into VPS table
- **vps_ip_history** — automatic IP change tracking via PostgreSQL trigger; records every `added`/`removed` IP with label, source, and timestamp
- **dns_accounts** — DNS provider accounts (Cloudflare, Route 53, GoDaddy, Name.com) with `provider_type` and JSONB `config`
- **dns_zones** — synced DNS zones per account, with domain/cert expiry tracking
- **dns_records** — synced DNS records with auto VPS-IP linking and provider-specific `extra` JSONB
- **api_keys** — multi-API-key management with per-key roles (admin/operator/readonly), SHA-256 hashed keys, `last_used_at` tracking, admin-only CRUD

### VPS fields of note:
- `ip_addresses` — JSONB array of `{ip, label}` objects. Labels: `china-telecom`, `china-unicom`, `china-mobile`, `china-cernet`, `overseas`, `internal`, `anycast`
- `purpose` — enum: vpn-exit, vpn-relay, vpn-entry, monitor, management, core-services
- `tags` — TEXT[] array with GIN index, for flexible categorization (e.g. optimized, premium, gpu)
- `extra` — JSONB for arbitrary metadata (cloud-synced VPS stores `cloud_instance_id`, `cloud_provider`, `cloud_region`)
- `source` — origin of the VPS record: `manual`, `agent`, `cloud-sync`
- `cloud_account_id` — FK to `cloud_accounts` table (NULL for manual/agent VPS)
- `monitoring_enabled` + `node_exporter_port` — controls whether this VPS appears in Prometheus targets

## Current State

### Backend (sigma-api) — Done:
- [x] Provider CRUD: `/api/providers`
- [x] VPS CRUD with filtering: `/api/vps` (filter by status, country, provider, purpose, tag, expiring_within_days)
- [x] IP addresses with carrier/type labels (JSONB: `{ip, label}`)
- [x] Quick retire endpoint: `POST /api/vps/{id}/retire`
- [x] Prometheus file_sd output: `GET /api/prometheus/targets`
- [x] Dashboard stats: `GET /api/stats`
- [x] Port allocation proxy: `POST /api/vps/{id}/allocate-ports` (forwards to agent)
- [x] IP change history: `GET /api/vps/{id}/ip-history` (auto-tracked via PostgreSQL trigger on `ip_addresses` column)
- [x] Multi-API-key management: `GET/POST /api/api-keys`, `GET/DELETE /api/api-keys/{id}` (per-key roles, SHA-256 hashed, admin-only CRUD, DB lookup in auth middleware with legacy env var fallback)
- [x] Dockerfile (multi-stage, rust:latest → debian:bookworm-slim)

### Frontend (sigma-web) — Done:
- [x] Dashboard: stats cards, charts (by country/status/provider), expiring VPS table
- [x] VPS list: filterable table (status/purpose/provider/country/tag), agent online/offline indicator, retire/delete actions
- [x] VPS form: create/edit with dynamic IP list + label selector (color-coded), read-only agent info panel (heartbeat, CPU, RAM, disk, uptime, load avg), port allocation button (find N available ports via agent)
- [x] VPS detail: IP history timeline (added/removed badges, paginated table)
- [x] Provider list + create/edit dialog
- [x] Settings page (API key config in localStorage)
- [x] Layout with sidebar navigation
- [x] Audit log page: filterable table (resource/action), expandable JSON details, admin-only
- [x] API Keys management page: create with one-time key display + copy, list with role badges, delete with confirmation, admin-only
- [x] Dockerfile (Node 20 → nginx:alpine, API reverse proxy)

### Deployment — Done:
- [x] Root-level docker-compose.yml (db + api + web)
- [x] Kubernetes configs (k8s/ directory)
- [x] GitHub Actions CI/CD workflows (per-package: api, web, agent, probe)
- [x] Makefile with common commands
- [x] DEPLOYMENT.md guide

### Roadmap:
- [x] CLI client (`sigma-cli/` — Rust binary using clap + reqwest)
- [x] Bulk import/export (CSV/JSON)
- [x] IP reachability check API (`/api/ip-checks` — CRUD, summary, purge)
- [x] IP reachability probe (`sigma-probe/` — ICMP/TCP/HTTP checks from China nodes)
- [x] VPS agent auto-registration + heartbeat (`sigma-agent/` + `/api/agent/*` endpoints)
- [x] Pagination on list endpoints
- [x] Telegram/webhook notifications for expiring VPS
- [x] Ansible dynamic inventory output (`GET /api/ansible/inventory`)
- [x] Cost tracking and reporting per provider/country/month
- [x] User authentication & RBAC (email+password, JWT, admin/operator/readonly roles)
- [ ] Auto-deploy node_exporter on new VPS via SSH
- [x] OpenAPI/Swagger spec generation (`/swagger-ui`, `/api-docs/openapi.json`)
- [x] Rate limiting (Redis-based sliding window, per-IP)
- [x] Tests
- [x] TOTP MFA (two-factor authentication with Google Authenticator / Authy)
- [x] Audit log (who changed what — tracks all mutations with user, action, resource, details)
- [x] Ticket system (issue tracking with status workflow, comments, priority, VPS/provider links)
- [x] Agent port scanning + Prometheus metrics (`/metrics` endpoint with port usage by process)
- [x] Agent port allocation (`POST /ports/allocate` on agent, proxied via API)
- [x] Grafana dashboard for port scan metrics (`grafana/dashboards/port-scan.json`)
- [x] Envoy xDS control plane (sigma-agent as gRPC xDS server, config stored in PostgreSQL, hot reload via LDS/CDS)
- [x] Envoy static config sync (agent parses `envoy.yaml` static_resources, syncs to API with `source=static`, topology shows dynamic vs static routes)
- [x] Multi-provider DNS management (Cloudflare, Route 53, GoDaddy, Name.com — read-only sync with VPS-IP linking, domain/cert expiry tracking)
- [x] Cloud account integration (AWS, Alibaba Cloud, DigitalOcean, Linode, Volcengine — store credentials, sync instances to VPS table, track source/origin)
- [x] VPS IP change history (PostgreSQL trigger on `ip_addresses` column auto-tracks added/removed IPs across all code paths — manual, agent, cloud sync, import)
- [x] eBPF TCP retransmit tracking + connection count monitoring (kprobes on `tcp_retransmit_skb`, `tcp_v4_connect`, `tcp_close`, `inet_csk_accept`)
- [x] eBPF UDP traffic monitoring (kprobe on `udp_sendmsg`/`udp_recvmsg` — covers WireGuard/Xray UDP transports)
- [x] eBPF TCP RTT/latency tracking (kprobe on `tcp_rcv_established` — per-process round-trip time for VPN quality)
- [x] eBPF packet drop monitoring (tracepoint `skb:kfree_skb` — drop counts with reason codes)
- [x] eBPF TCP connection latency (SYN-to-established time via `tcp_v4_connect` entry+return)
- [x] eBPF DNS query tracing (trace UDP sends to port 53 — detect DNS leaks on VPN nodes)
- [ ] eBPF disk I/O per process (kprobe on `vfs_read`/`vfs_write` — identify disk bottlenecks)
- [x] eBPF OOM kill tracking (tracepoint `oom:mark_victim` — immediate process kill alerts)
- [ ] eBPF socket buffer depth monitoring (track `sk_rcvbuf`/`sk_sndbuf` usage — detect buffer bloat)
- [ ] eBPF outbound connection audit (log all `connect()` with dest IP/port — detect unexpected traffic)
- [x] eBPF exec tracing (tracepoint `sched:sched_process_exec` — intrusion detection on remote VPS)

## Tech Stack

### Backend
- Rust 1.88+ (uses `rust:latest` in Docker)
- Axum 0.8, Tokio, SQLx 0.8 (with `json`, `rust_decimal` features)
- PostgreSQL 16

### Frontend
- Vite 7, React 19, TypeScript
- Tailwind CSS v4 (via `@tailwindcss/vite` plugin)
- React Router v7, TanStack React Query v5
- React Hook Form, Axios, Recharts, Lucide React

## Code Conventions

- Each route module exposes `pub fn router() -> Router<AppState>`
- Error handling via `AppError` enum → JSON error responses
- All handlers return `Result<Json<T>, AppError>`
- IP addresses stored as JSONB `[{ip, label}]`, validated server-side via `std::net::IpAddr`
- Partial updates: PUT endpoints fetch existing record, merge with provided fields
- Frontend: React Query hooks in `src/hooks/`, API layer in `src/api/`, types in `src/types/api.ts`
- Auth: JWT in localStorage (`sigma_token`), AuthContext provides `useAuth()` hook, ProtectedRoute component for route guarding. API keys via `X-Api-Key` header — DB-managed keys checked first (SHA-256 hash lookup), legacy `API_KEY` env var as fallback
- RBAC: four roles with descending privilege. API keys respect role checks (no bypass)

### RBAC Role Matrix

| Scope | admin | operator | agent | readonly |
|-------|:-----:|:--------:|:-----:|:--------:|
| All GET endpoints (read) | ✓ | ✓ | ✓ | ✓ |
| VPS / Provider / DNS / Cloud CRUD | ✓ | ✓ | — | — |
| Tickets / IP Checks / Costs / Import | ✓ | ✓ | — | — |
| `POST /agent/register`, `/agent/heartbeat` | ✓ | ✓ | ✓ | — |
| Envoy nodes & routes write (8 endpoints) | ✓ | ✓ | ✓ | — |
| User management (`/api/users`) | ✓ | — | — | — |
| API key management (`/api/api-keys`) | ✓ | — | — | — |
| Audit logs, system settings | ✓ | — | — | — |

## Build & Run

```bash
# Full stack via Docker Compose (from project root)
docker compose up -d

# Access:
#   Web UI:  http://localhost
#   API:     http://localhost:3000/api/stats
#   DB:      localhost:5432 (sigma/sigma)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lai3d)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lai3d)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
