## claude-yolo

> This file provides guidance to Claude Code when working in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project Overview

Docker-based Claude Code environment with safety features (secrets scanning, git hooks, command logging). Designed to protect both the host machine AND inexperienced users (sales engineers, FDEs, CTOs) from well-intentioned but potentially dangerous actions, while giving Claude Code maximum autonomy.

## Available Tools

**Core:** git, gh, jq, ripgrep (rg), vim, nano, tmux, htop, tree
**Cloud:** aws, az, gcloud
**Kubernetes:** kubectl, helm, k9s, docker, docker-compose
**IaC:** terraform, tfsec
**Databases:** psql, mysql, redis-cli, mongosh
**Python:** uv (primary package manager), ruff, mypy, pytest, bandit, pre-commit
**Security:** gitleaks, detect-secrets, trivy
**Networking:** tailscale, openvpn, cloudflared, ttyd
**Utilities:** yq, httpie (http), curl, wget
**Build:** make, cmake, gcc/g++, Node.js 20

## File Structure

```
claude-yolo/
├── src/claude_yolo/           # Python package (CLI tool)
├── terraform/                 # Infrastructure as Code
│   └── azure/                # Azure deployment modules
│       ├── acr/              # Azure Container Registry
│       ├── aci/              # Azure Container Instances
│       ├── vm/               # Azure Virtual Machines
│       └── scripts/          # Helper scripts (push-to-acr.sh)
├── examples/                  # Demo applications
│   └── fastapi-hello-world/  # Simple FastAPI demo
├── demos/                     # Sales engineering demos
│   └── sales-engineering/    # SE demo scripts and guides
├── docs/                      # Documentation
├── tests/                     # Test suite
└── .github/workflows/         # CI/CD pipelines
```

## Safety Constraints

**Container Isolation:**
- Runs as non-root user (developer, UID 1001)
- Resource limits: 2 CPU cores, 4GB RAM (configurable via `.env`)
- Network isolation via Docker bridge
- No Docker-in-Docker access

**Git Safety Hooks:**
- Pre-commit: Scans for secrets (gitleaks, detect-secrets), large files, sensitive patterns
- Pre-push: Prevents force push to main/master, warns on direct pushes
- Auto-configured via `core.hooksPath`

**Pre-commit Framework:**
- Secrets detection (gitleaks, detect-secrets)
- Python linting and formatting (ruff)
- Security scanning (bandit)
- File safety checks (large files, merge conflicts)

**Logging:**
- `/logs/commands/` - All shell commands with timestamps
- `/logs/claude/` - Claude Code session logs
- `/logs/git/` - Git operations
- `/logs/safety/` - Safety check results
- All logs shared with host via volume mount

**Claude Code Mode:**
- BypassPermissions mode enabled by default (`config/.claude/settings.local.json`)
- True YOLO mode - autonomous operation without permission prompts
- Safety provided by container isolation and hooks

## Networking Modes

The container supports two networking modes in `docker-compose.yml`. **Claude should recommend the appropriate mode based on user needs:**

### Mode 1: Bridge Networking (Default)
- **When to recommend:** Multi-container setups (databases, Redis, microservices)
- **Pros:** Docker DNS, network isolation, can join custom networks
- **Cons:** MCP OAuth callbacks won't work (requires manual port forwarding)
- **Config:** Uses `networks: - claude-network` (default in docker-compose.yml)

### Mode 2: Host Networking (For MCP OAuth)
- **When to recommend:** Single-container setup + user needs MCP server authentication (Atlassian, GitHub MCP servers)
- **Pros:** All ports accessible, MCP OAuth works seamlessly, zero overhead
- **Cons:** Cannot join Docker networks, no service discovery, multi-container setups won't work
- **Config:** Use `claude-yolo run --mcp` flag (automatically applies host networking)
  - **Advanced:** Manually edit docker-compose.yml - uncomment `network_mode: "host"` and comment out `networks:`

### MCP OAuth Technical Background
Claude Code uses random ephemeral ports (49152-65535) for OAuth callbacks. Exposing this full range causes:
- Container startup hangs (hours) or complete failure
- 16GB+ RAM consumption
- Docker creates 16,384 iptables rules or docker-proxy processes

**Decision tree for users:**
- Planning to add databases/microservices? → Bridge mode (default)
- Need MCP OAuth + single container only? → Host mode
- Unsure? → Start with bridge mode (default)

See [GitHub Issue #2527](https://github.com/anthropics/claude-code/issues/2527) for Claude Code's OAuth port limitation and [Docker Issue #14288](https://github.com/moby/moby/issues/14288) for port range performance issues.

## File Boundaries

**Safe to edit:** `/workspace/*`, `/config/*`, `/scripts/*`, `Dockerfile`, `docker-compose.yml`, `.env`
**Read-only:** `/opt/config-templates/*`, `/logs/*` (view only)
**Never touch:** System directories, volume mount points

## Volume Mounts

- `/home/developer` - Host home directory bind mount (configurable via `HOST_HOME`, default: `./home`)
- `/workspace` - Host workspace directory (configurable via `HOST_WORKSPACE`)
- `/logs` - Shared logs directory (configurable via `HOST_LOGS`)
- `/mnt/host-gitconfig` - Optional mount for host git config

**Note:** The home directory uses a bind mount for transparency and easy access to configs. Files created by the container are owned by UID 1001 (container user). On macOS/Windows Docker Desktop, this is handled transparently. On Linux, you may need to adjust ownership with `sudo chown -R $USER ./home` if needed.

## Common Commands

```bash
# Set up safety features for a new project
/home/developer/scripts/setup-project-safety.sh /workspace/my-project

# View logs
tail -f /logs/safety/checks.log
tail -f /logs/git/operations.log

# Python projects (use uv as primary package manager)
uv init .
uv add <package>
```

## Development Approach

When working in this repository:

1. **Infrastructure Focus**: Changes typically involve Dockerfile, docker-compose, or shell scripts
2. **Safety First**: Every new capability should consider security implications
3. **User Protection**: Target users may be inexperienced - protect them from foot-guns
4. **Balance**: Maximum Claude Code autonomy within safety constraints
5. **Logging**: Ensure new features log appropriately for transparency
6. **Documentation**: Keep CLAUDE.md and README.md updated

IMPORTANT: This environment is designed for users who may be disconnected from architecture details. Always prioritize safety and transparency.

## Target Users

Sales Engineers (demoing safely), Forward-Deployed Engineers (customer environments), CTOs/Leadership (experimenting with Claude Code), Anyone wanting maximum autonomy with maximum safety.

## Azure Infrastructure (New!)

OpenTofu/Terraform modules for deploying claude-yolo to Microsoft Azure. All modules are compatible with both OpenTofu (recommended) and Terraform.

**Available Modules:**
- **ACR (Azure Container Registry):** Private Docker registry for claude-yolo images
  - Path: `terraform/azure/acr/`
  - Cost: ~$5/month (Basic tier)
  - Deploy time: ~3 minutes

- **ACI (Azure Container Instances):** Serverless container deployment
  - Path: `terraform/azure/aci/`
  - Cost: ~$30-40/month (2 vCPU, 4GB RAM)
  - Deploy time: ~5 minutes
  - Best for: Demos, development, quick testing

- **VM (Virtual Machines):** Ubuntu VM with Docker and auto-deployment
  - Path: `terraform/azure/vm/`
  - Cost: ~$30-70/month (depends on size)
  - Deploy time: ~10 minutes
  - Best for: Traditional deployments, persistent workloads

**Quick Deploy Workflow:**
```bash
# 1. Deploy ACR
cd terraform/azure/acr
tofu init && tofu apply

# 2. Push image to ACR
cd ../scripts
./push-to-acr.sh --terraform-dir ../acr --project-dir /path/to/project

# 3. Deploy to ACI or VM
cd ../aci  # or ../vm
tofu init && tofu apply
```

**Demo Resources:**
- 5-minute sales demo: `demos/sales-engineering/5-minute-demo.md`
- Azure quick demo script: `demos/sales-engineering/azure-quick-demo.sh`
- Example app: `examples/fastapi-hello-world/`
- Azure quickstart guide: `docs/azure-quickstart.md`

**Future Cloud Support:**
- AWS (ECS, ECR, EC2) - Planned next
- GCP (GCE, GCR, GKE) - Roadmap
- Multi-cloud (Kubernetes, Nomad) - Future

See `terraform/azure/README.md` for comprehensive documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heysamtexas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
