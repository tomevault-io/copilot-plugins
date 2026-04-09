## barf-docker

> Docker Compose orchestration for [barf](../barf-ts) — an AI issue/work management CLI that dispatches work to Claude.

# barf-docker

Docker Compose orchestration for [barf](../barf-ts) — an AI issue/work management CLI that dispatches work to Claude.

## Architecture

```
┌─────────────┐   docker exec    ┌──────────────┐
│  barf        │ ──────────────→ │ claude-cli-1  │
│  (bun/ts)    │                 └──────────────┘
│              │   docker exec    ┌──────────────┐
│              │ ──────────────→ │ claude-cli-2  │
└─────────────┘                  └──────────────┘
```

- **barf** container runs the TypeScript CLI, has Docker socket mounted to orchestrate claude-cli containers
- **claude-cli-N** containers run `claude` CLI with separate auth volumes (separate team accounts)
- **claude-wrapper.sh** inside barf acts as a `claude` shim — discovers containers by label, filters healthy/rate-limited, picks least-busy
- **dozzle** provides log viewing at `http://localhost:8080`

## Container Discovery

Containers are discovered via Docker label `barf.role=claude-cli`, not by name. All claude-cli services must have this label.

## Commands

```bash
# Single account (default)
docker compose up -d

# Dual account
docker compose --profile dual up -d

# Login to Claude on each container
docker compose exec -it claude-cli-1 claude login
docker compose --profile dual exec -it claude-cli-2 claude login

# Run barf command
docker compose run barf auto

# Check health
docker compose ps

# Generate compose for N accounts
./generate-compose.sh 3 > docker-compose.yml
```

## Key Files

| File | Purpose |
|------|---------|
| `barf/claude-wrapper.sh` | Smart dispatch shim (health, rate-limit, least-busy) |
| `barf/barf-entrypoint.sh` | Entrypoint: sleep (exec mode) or run barf (args mode) |
| `claude-settings.json` | Shared Claude settings stub, mounted read-only |
| `generate-compose.sh` | Generate docker-compose.yml for N accounts |
| `.env` | `PROJECT_DIR` and `BARF_TS_DIR` paths |

## Environment Variables

- `PROJECT_DIR` — host directory mounted into claude-cli containers at `/workspace`
- `BARF_TS_DIR` — path to barf-ts source, mounted into barf container at `/app`
- `ANTHROPIC_API_KEY` — optional, passed to barf container for SDK calls

## Conventions

- Pin base images to minor versions (e.g., `oven/bun:1.2`, not `latest`)
- Use official Docker CE CLI apt repo (not `docker.io` package) — avoids API version mismatches
- `procps` is required in claude-cli for `pgrep` (used by wrapper for process counting)
- Wrap `pgrep` output in `sh -c` and strip non-numeric chars to avoid multiline/formatting issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vx-daniel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vx-daniel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
