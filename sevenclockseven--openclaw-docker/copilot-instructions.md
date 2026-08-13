## openclaw-docker

> This document provides guidelines for AI agents working in the OpenClaw Docker repository.

# OpenClaw Docker AGENTS.md

This document provides guidelines for AI agents working in the OpenClaw Docker repository.

## Project Overview

OpenClaw is a Docker-based deployment of Claude AI with gateway and CLI services. The repository contains Docker configuration for containerized Claude AI services.

## Build Commands

```bash
# Build Docker image
docker build -t openclaw-cn:local .

# Build with custom APT packages
docker build --build-arg OPENCLAW_DOCKER_APT_PACKAGES="pkg1 pkg2" -t openclaw-cn:local .

# Build and run services
docker-compose up -d

# Build specific service only
docker-compose build openclaw-cn-gateway
```

## Runtime Commands

```bash
# Start gateway service (binds to lan by default)
docker-compose up -d openclaw-cn-gateway

# Start CLI service
docker-compose up -d openclaw-cn-cli

# Run CLI interactively
docker-compose run --rm openclaw-cn-cli

# View logs
docker-compose logs -f openclaw-cn-gateway
docker-compose logs -f openclaw-cn-cli

# Stop services
docker-compose down
```

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `OPENCLAW_GATEWAY_TOKEN` | Gateway authentication token | No (generates if missing) |
| `CLAUDE_AI_SESSION_KEY` | Claude AI session key | Yes (for CLI/gateway) |
| `CLAUDE_WEB_SESSION_KEY` | Web session key | Optional |
| `CLAUDE_WEB_COOKIE` | Web cookie | Optional |
| `OPENCLAW_GATEWAY_BIND` | Binding interface (`lan`, `127.0.0.1`) | No (default: lan) |
| `OPENCLAW_GATEWAY_PORT` | Gateway port | No (default: 18789) |
| `OPENCLAW_BRIDGE_PORT` | Bridge port | No (default: 18790) |

## Code Style Guidelines

### Docker Best Practices

- **Multi-stage builds**: Use when possible for smaller images
- **Security**: Run containers as non-root user (`node` user, uid 1000)
- **Minimal base image**: Use `node:22-bookworm-slim` for production
- **Layer caching**: Order Dockerfile instructions from least to most changing
- **No hardcoded secrets**: Use environment variables only

### Dockerfile Conventions

```dockerfile
# Use specific version tags, never :latest
FROM node:22-bookworm

# Combine related RUN commands to reduce layers
RUN apt-get update && \
    apt-get install -y --no-install-recommends pkg && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Always set working directory
WORKDIR /app

# Use COPY for static files, ADD for archives
COPY package.json pnpm-lock.yaml ./

# Use non-root user for security
RUN chown -R node:node /app
USER node

# Use exec form for CMD/ENTRYPOINT
CMD ["node", "dist/index.js", "gateway"]
```

### docker-compose.yml Conventions

```yaml
services:
  app:
    image: ${IMAGE:-app:local}
    environment:
      - VAR=value  # Array syntax preferred
    volumes:
      - ${CONFIG_DIR}:/home/node/.app
    ports:
      - "${HOST_PORT}:CONTAINER_PORT"
    restart: unless-stopped
    init: true  # Use init: true instead of tini
```

### Shell Scripting

```bash
#!/bin/bash
set -euo pipefail  # Always use strict mode

# Use quotes around variable expansions
"$VAR"

# Check for required variables
: "${REQUIRED_VAR:?ERROR: REQUIRED_VAR is required}"
```

## Type Safety

- Use TypeScript for all application code
- Enable `strict: true` in tsconfig.json
- Never use `any`, `@ts-ignore`, or type assertions to suppress errors
- Use proper error types and validation

## Error Handling

- Use typed errors with descriptive messages
- Log errors with context before throwing
- Never leave empty catch blocks
- Handle promise rejections explicitly

## Import Conventions

```typescript
// Absolute imports for internal modules
import { something } from '@/module/path'

// Relative imports for parent/peer directories
import { helper } from '../utils/helper'

// Group imports by type: external, internal, relative
import fs from 'node:fs'
import path from 'node:path'
import { something } from '@/internal/module'
import { local } from './local-file'
```

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files | kebab-case | `my-config-file.ts` |
| Classes | PascalCase | `GatewayServer` |
| Functions/variables | camelCase | `startGateway()` |
| Constants | UPPER_SNAKE_CASE | `DEFAULT_PORT` |
| Interfaces | PascalCase (no I prefix) | `UserConfig` |
| Docker images | kebab-case | `openclaw-cn` |

## Testing

```bash
# Run tests in container
docker-compose run --rm openclaw-cn-cli npm test

# Run specific test
docker-compose run --rm openclaw-cn-cli npm test -- --testNamePattern="gateway"

# View test coverage
docker-compose run --rm openclaw-cn-cli npm test -- --coverage
```

## Git Workflow

- **Commits**: Use conventional commits (`feat:`, `fix:`, `docs:`, `chore:`)
- **Branches**: `feature/*`, `fix/*`, `docs/*`
- **Pull requests**: Required for main branch changes
- **Never commit**: `node_modules/`, `dist/`, `.env*` (except `.env.example`)

## Container Security

- Never run as root in production
- Use read-only volumes where possible
- Set resource limits in docker-compose
- Use specific image tags (never `:latest`)
- Scan images for vulnerabilities: `docker scan openclaw-cn:local`

---
> Source: [sevenclockseven/openclaw-docker](https://github.com/sevenclockseven/openclaw-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
