## ai-swarm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Swarm is an autonomous development orchestration system powered by LLMs and Temporal.io. Unlike simple AI coding assistants, AI Swarm manages the entire software development lifecycle—from planning and coding to review and deployment—with durable execution, automatic retries, and Git branch isolation.

## Development Commands

```bash
# Root level (turbo monorepo)
pnpm dev          # Start all services in development mode
pnpm build        # Build all packages
pnpm test         # Run tests across all packages
pnpm lint         # Lint all code
pnpm format       # Format with Prettier

# Individual services
pnpm worker       # Start worker service (packages/worker)
pnpm portal       # Start portal dashboard (apps/portal)
pnpm cli          # Start CLI tool (apps/cli)

# Docker-based development (recommended for local testing)
docker compose up                       # Start all services
docker compose -f docker-compose.yml -f docker-compose.local.yml up -d  # Development mode
docker logs -f ai-swarm-worker-1        # View worker logs
docker logs -f ai-swarm-portal          # View portal logs
```

## High-Level Architecture

AI Swarm follows a **decoupled, event-driven architecture** designed for reliability and scalability:

### Core Components

1. **Temporal.io Orchestration** (`packages/workflows/`)
   - The "brain" of the system—manages state persistence, timeouts, and retries
   - Workflows durable across server restarts
   - Main workflow: `developFeature` in `packages/workflows/src/workflows/develop-feature.ts`

2. **Stateless Workers** (`packages/worker/`)
   - Execute Temporal activities (planning, coding, testing, deployment)
   - Ephemeral and horizontally scalable (1-8 workers via `./scale-workers.sh`)
   - Entry point: `packages/worker/src/index.ts`

3. **Portal UI** (`apps/portal/`)
   - Next.js 14 dashboard for task submission and workflow monitoring
   - Real-time status updates via Redis pub/sub
   - Passkey (WebAuthn) authentication—no external IdP dependencies

4. **Shared Package** (`packages/shared/`)
   - Type definitions, LLM client, database client, services
   - SCM integration (GitHub, GitLab, Azure DevOps)
   - Context discovery for `.aicontext/` and `claude.md` files

### Services Architecture (Docker)

```
┌─────────────────────────────────────────────────────────────────┐
│                        AI Swarm Stack                            │
├─────────────────────────────────────────────────────────────────┤
│  temporal          ──  Workflow orchestration engine             │
│  temporal-ui       ──  Workflow visualization UI                  │
│  postgres          ──  Temporal persistence                      │
│  redis             ──  Caching and worker health monitoring       │
│  worker (scaled)   ──  Temporal worker processes (1-8 replicas)  │
│  portal            ──  Next.js dashboard UI                      │
│  builder           ──  Persistent build environment               │
│  playwright        ──  Browser testing service                   │
│  socket-proxy      ──  Secure Docker socket access               │
└─────────────────────────────────────────────────────────────────┘
```

## Workflow Lifecycle

Every task follows a rigorous multi-stage pipeline (defined in `packages/workflows/src/workflows/develop-feature.ts`):

1. **Planning** (`planTask`) — Analyzes codebase context and decomposes task into actionable steps
2. **Worktree Creation** (`createWorktree`) — Isolated Git worktree for safe development
3. **Coding** (`executeCode`) — Implements changes with atomic self-correction (up to 3 attempts)
4. **Shadow Review** (`reviewCode`) — Autonomous code review against plan and standards
5. **Build Verification** (`verifyBuild`) — Runs build and tests in isolated environment
6. **PR Merge** (`mergePullRequest`) — Squash merge to main with branch deletion
7. **Deployment** (`deployToProduction`) — LLM-powered intelligent deployment with rollback on failure
8. **Post-Deploy Verification** (`verifyDeployment`) — Live verification of production deployment

## LLM Integration

AI Swarm uses a **multi-role LLM strategy** with model cascades:

| Role | Default Provider | Models | Purpose |
|------|------------------|--------|---------|
| `planner` | Gemini | `gemini-2.5-pro`, `gemini-2.5-flash` | Deep reasoning for task planning |
| `coder` | Claude/Gemini | Configurable | Code implementation |
| `reviewer` | Gemini | `gemini-2.5-pro` | Code review and quality checks |
| `deployer` | Gemini | `gemini-2.5-flash`, `gemini-2.5-pro` | Deployment troubleshooting |
| `portal_planner` | Gemini | `gemini-2.5-flash` | Fast chat responses in Portal |

**Configuration:** Set `LLM_{ROLE}=claude|gemini` environment variable per role. Defaults to Gemini for most roles.

**Claude Integration:** Uses Z.ai API proxy (`Z_AI_API_KEY`). OAuth credentials shared via `workers_oauth` volume.

## Key Patterns

### 1. Temporal Activities

Activities in `packages/workflows/src/activities/` are **individual units of work** executed by workers:

- `planner.ts` — `planTask`, `reviewCode`
- `coder.ts` — `executeCode`
- `deployer.ts` — `verifyBuild`, `deployToProduction`, `verifyDeployment`
- `rollback.ts` — `rollbackCommit`, `createFixTask`
- `worktree-manager.ts` — `createWorktree`, `removeWorktree`
- `builder.ts` — `runInBuilder` (execute commands in builder container)
- `playwright-runner.ts` — `runPlaywrightTest`, `captureScreenshotAsBase64`
- `llm-deployer.ts` — `analyzeDeploymentContext`, `troubleshootDeployment`, `executeRecoveryAction`

**Activity timeout:** 15 minutes with exponential backoff retry.

### 2. Context Discovery

The system automatically discovers project context from:
- `.aicontext/` folder (project-specific documentation)
- `claude.md` file (global project instructions)
- `docs/context` folder (legacy support)
- `ai-swarm.deploy.yaml` (declarative deployment configuration)

Implementation: `packages/shared/src/context-discovery.ts`

### 3. Git Worktree Isolation

All development happens in **isolated Git worktrees** to avoid polluting the main branch:

- Worktree path pattern: `{workspace}/.worktrees/task-{id}/`
- Automatic cleanup on success/failure
- Supports multi-project workspaces

Implementation: `packages/workflows/src/activities/worktree-manager.ts`

### 4. Self-Healing Infrastructure

**Fix Task Loop:** When deployment fails, a "fix task" is automatically created with error context. The system detects loops (repeated failures) and escalates to human after 3 attempts.

**LLM Deployer (v3.0.0):** Uses LLM to classify deployment errors:
- **Code errors** → Create fix task, send back to coder
- **Infrastructure errors** → Attempt automatic recovery (restart container, clear cache, etc.)
- **Unknown errors** → Retry with backoff, escalate after 3 attempts

### 5. Multi-Project Support

Each project in the Portal has:
- Unique `projectId` for SCM credential lookup
- Separate deployment configuration
- Independent LLM role assignments
- Workspace root: `${WORKSPACE_ROOT:-./workspace}`

## Important Files

| File | Purpose |
|------|---------|
| `packages/workflows/src/workflows/develop-feature.ts` | Main workflow orchestration |
| `packages/workflows/src/activities/` | All Temporal activities |
| `packages/shared/src/llm.ts` | LLM invocation with model cascade |
| `packages/shared/src/types.ts` | Core type definitions |
| `packages/shared/src/context-discovery.ts` | Project context discovery |
| `apps/portal/src/lib/temporal.ts` | Temporal client connection |
| `docker-compose.yml` | Base Docker services definition |

## Deployment Configuration

Projects can declare deployment configuration via `ai-swarm.deploy.yaml` in their source root:

```yaml
build: pnpm run build
deploy: docker compose up -d --build
verify: curl -f https://example.com/health || exit 1
```

Implementation: `packages/shared/src/services/DeployConfigService.ts`

## Environment Variables

Key environment variables (see `docker-compose.yml` and `./setup.sh`):

- `Z_AI_API_KEY` — Z.ai API key for Claude access
- `TEMPORAL_ADDRESS` — Temporal server address (default: `temporal:7233`)
- `REDIS_URL` — Redis connection string
- `PROJECT_DIR` — Path to project source code
- `WORKSPACE_ROOT` — Multi-project workspace root
- `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_DIR` — SSH deployment targets
- `LLM_PLANNER`, `LLM_CODER`, `LLM_REVIEWER` — Per-role LLM provider selection

## Testing

```bash
# Run tests for a specific package
pnpm --filter @ai-swarm/shared test
pnpm --filter @ai-swarm/workflows test

# Run with coverage (vitest)
pnpm --filter @ai-swarm/workflows test --coverage
```

## Troubleshooting

**Worker not processing tasks:**
- Check Temporal UI: `${PORTAL_DOMAIN}/temporal`
- Verify worker health: `docker logs ai-swarm-worker-1`
- Check Redis heartbeat: `docker exec ai-swarm-redis redis-cli KEYS "worker:*"`

**LLM timeout:**
- Default timeout: 10 minutes per model attempt
- Check network connectivity to LLM providers
- Verify `Z_AI_API_KEY` is set for Claude

**Build failures:**
- Check builder container logs: `docker logs ai-swarm-builder`
- Ensure `PROJECT_DIR` is correctly mounted
- Verify build tools are installed in builder

## Release Process

Images are published to GHCR on GitHub release creation:

```bash
# Create a new release (triggers GitHub Actions)
git tag -a v3.0.2 -m "Release v3.0.2"
git push origin v3.0.2
```

Images are built for both AMD64 and ARM64 using native runners (no QEMU emulation).

## Version History

- **v3.0.1** (2026-01-09) — Claude CLI process timeout fix, GHCR automation, multi-arch builds
- **v3.0.0** (2026-01-06) — Complete rewrite with Temporal.io, sovereign auth, multi-project support
- **v2.1.0** (2026-01-02) — Initial public release

---
> Source: [ai-swarm-dev/ai-swarm](https://github.com/ai-swarm-dev/ai-swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
