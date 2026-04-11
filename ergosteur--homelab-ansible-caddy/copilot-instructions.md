## homelab-ansible-caddy

> This project manages the deployment of a custom Caddy reverse proxy using Ansible. It builds a custom Caddy binary with specific plugins (`xcaddy`) and deploys it to target hosts defined in the inventory.

# Project Context: Homelab Ansible Caddy

## Overview
This project manages the deployment of a custom Caddy reverse proxy using Ansible. It builds a custom Caddy binary with specific plugins (`xcaddy`) and deploys it to target hosts defined in the inventory.

## Architecture
- **Build Host:** The machine running Ansible (localhost or a dedicated controller). Compiles the Caddy binary.
- **Target Hosts:** Servers (e.g., `caddy-rproxy1`, `caddy-rproxy2`) where Caddy is deployed.
- **Orchestration:** Ansible Playbooks (`site.yml`) manage the state of the target hosts and perform post-deployment smoke tests.
- **Environment:** Uses a Python virtual environment (`~/ansible-venv`) for Ansible and `yq` for inventory parsing.
- **Configuration:** 
  - `ansible.cfg`: Centralized Ansible settings (inventory path, SSH tuning).
  - `ansible/group_vars/`: Optimized variable storage separated by group. Use `sample_*.yml` as templates.
  - `ansible/inventory/`: Clean inventory files. Use `sample_hosts.yml` as a template.

## Key Components

### 1. Build System (`Makefile`)
- Wraps `xcaddy` and `ansible-playbook`.
- `make build`: Compiles `caddy` with plugins (Cloudflare DNS, RealIP, Cache, Transform Encoder).
- `make deploy`: Runs the Ansible playbook.
- `make test`: Runs only the verification tasks and generates `test_report.csv`.
- `make ssh-bootstrap`: Helper to copy SSH keys to all hosts in the inventory using `yq`.
- `make update`: Local installation helper to replace `/usr/bin/caddy` and restart the service.

### 2. Ansible Role (`ansible/playbooks/roles/caddy`)
- **Templates:**
  - `Caddyfile.j2`: The core logic. Generates the Caddy configuration based on `caddy_sites` variable.
  - `caddy.service`: Systemd unit file.
- **Tasks:**
  - `main.yml`: User setup, binary distribution, config generation, service management.

### 3. Configuration (`ansible/inventory/hosts.yml`)
- Defines `caddy_sites` list.
- Supports advanced routing and features:
  - **Reverse Proxy:** Simple `upstream` or multiple path-based `upstreams`.
  - **Wildcard Subdomains:** `subdomain_upstreams` for dynamic routing.
  - **Redirects:** `redirect` block for domain/path redirection (301/302).
  - **Caching:** `cache_static` and `cache_ttl` using the `cache-handler` plugin.
  - **Static Files:** `serve_static` and `root_path` for hosting local content.
  - **TLS Options:** `tls_internal` for local CA or Cloudflare DNS-01 (default).
  - **HTTPS Upstreams:** `upstream_sni` and `skip_upstream_cert_verify` for secure backends.

## Recent Changes (March 2026)
- **Verification Tests:** Added `post_tasks` to `site.yml` and a dedicated `test.yml` playbook. They perform smoke tests on all `caddy_sites` to ensure they are responding, and generate a `test_report.csv` file with results.
- **Feature Consolidation:** Documented all major Caddyfile features including redirects, caching, and static file serving.
- **HTTPS Upstream Support:** (Jan 2026) Updated `Caddyfile.j2` to automatically handle SNI when the upstream target is `https://`.
- **Infrastructure:** Added `ssh-bootstrap` to `Makefile` to simplify initial setup of new proxy nodes.

## Developer Notes
- **Git:** Project is version controlled. `caddy` binary is ignored.
- **Safety:** Always verify `Caddyfile` generation with `ansible-playbook --check` or by inspecting the generated file on a test host before full rollout.
- **Plugins:** Custom binary includes `cloudflare`, `realip`, `cache-handler`, and `transform-encoder`.
- **Default Sites:** The Ansible role automatically adds two default internal sites for each host (hostname and FQDN) serving a test page at `/var/www/caddy-test`.
- **Secrets:** Cloudflare API tokens are stored in `/etc/caddy/cloudflare.env` on target hosts.
- **Internal TLS:** Requires `libnss3-tools` (installed by the role) for managing local trust stores if using `tls_internal`.
- **Logs:** Caddy logs are stored in `/var/log/caddy/` with rotation and custom formatting.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ergosteur)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ergosteur)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
