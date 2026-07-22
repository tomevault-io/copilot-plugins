## kanban-loader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Communication Language

**日本語でコミュニケーションを行ってください。** This project's primary communication language is Japanese. Please communicate in Japanese.

## Project Overview

kanban-loader is a Docker-based development environment that integrates vibe-kanban (AI development task management tool) with development tools including Claude Code, GitHub CLI, Git, and Node.js 22. The environment uses 1Password SSH Agent for authentication.

## Architecture

### Container Structure
- **Base Image**: node:22-bookworm-slim
- **User Setup**:
  - `kanban` user (UID 1001, GID 1000) - primary development user
  - `node` user (UID/GID 1000) - shared group for Node.js operations
- **Entry Point**: `/entrypoint.sh` - handles directory setup, permissions, and user switching

### Key Components
1. **vibe-kanban**: Runs as the main service via `npx vibe-kanban` on port 3000 (exposed as 4989)
2. **Claude Code**: Installed globally via npm as `@anthropic-ai/claude-code`
3. **GitHub CLI**: System package for GitHub operations
4. **SSH Agent Integration**: 1Password socket mounted at `/ssh-agent`

### Volume Architecture
- `repos-volume`: Named volume for cloned repositories at `/repos`
- `./data/vibe-kanban`: Persistent vibe-kanban shared data
- `vibe-kanban-cache`, `vibe-kanban-tmp`: Cache and temporary storage
- `claude-config`, `gh-config`: Tool configurations
- Host-mounted volumes: `.gitconfig` (read-only), 1Password SSH socket

## Common Commands

### Docker Operations
```bash
# Start the environment
docker compose up -d

# Access container as kanban user (primary development)
docker compose exec -u kanban app bash

# Access as root (for system operations)
docker compose exec app bash

# Check logs (especially useful for verifying vibe-kanban startup)
docker compose logs -f app

# Stop/restart
docker compose down
docker compose restart

# Rebuild after configuration changes
docker compose up -d --build
```

### Inside Container

#### vibe-kanban
- Server runs on http://localhost:3000 (host: http://localhost:4989)
- Wait for log message "Server running on http://0.0.0.0:3000" before accessing
- Update: `npm update -g vibe-kanban` (as root), then restart container

#### Claude Code
```bash
# First-time setup (requires API key)
claude

# Normal usage
claude
```

#### GitHub CLI
```bash
# Initial authentication
gh auth login

# Check status
gh auth status
```

#### Repository Management
```bash
# Working directory
cd /repos

# Clone repositories (first time requires accepting SSH fingerprint)
git clone git@github.com:username/repository.git
```

## Development Workflow

1. **Initial Setup** (inside container):
   - Run `claude` for first-time API key setup
   - Run `gh auth login` for GitHub authentication
   - SSH keys via 1Password are automatically available

2. **Repository Work**:
   - All repositories should be cloned into `/repos`
   - Git config is read from host `.gitconfig`
   - SSH authentication uses 1Password agent at `$SSH_AUTH_SOCK=/ssh-agent`

3. **Customization**:
   - Add packages to `config/apt.txt` (one per line, rebuild container)
   - Override ports via `compose.override.yml`
   - Persistent data survives container recreation in `./data/`

## Important Notes

### Permissions
- The entrypoint script (`entrypoint.sh`) handles permission setup for mounted volumes
- All development work should be done as `kanban` user
- System operations require root access

### Environment Variables
- `NPM_CONFIG_PREFIX=/home/kanban/.npm-global`
- `PATH` includes `/home/kanban/.npm-global/bin`
- `SSH_AUTH_SOCK=/ssh-agent` (1Password integration)
- `DOCKER_BUILDKIT=1` (Docker build optimization)

### Data Persistence
- All data in `./data/` directory persists across container recreation
- `repos-volume` is a named volume - use `docker volume` commands to manage
- Claude and GitHub CLI configurations are stored in dedicated volumes

### Platform
- Built for `linux/amd64` (specified in compose.yml)
- Uses Debian Bookworm base (from node:22-bookworm-slim)

## Troubleshooting

### SSH Issues
- Verify `$SSH_AUTH_SOCK=/ssh-agent`
- Ensure 1Password SSH Agent is enabled
- Check SSH keys are registered in 1Password

### Permission Issues
- Check `/entrypoint.sh` execution logs
- Verify ownership: `ls -la ~/.claude.json`, `ls -la ~/.config/gh`

### vibe-kanban Startup
- Monitor logs for "Server running" message
- Health check targets http://localhost:3000
- Allow 30-60 seconds for full initialization

---
> Source: [ms2sato/kanban-loader](https://github.com/ms2sato/kanban-loader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
