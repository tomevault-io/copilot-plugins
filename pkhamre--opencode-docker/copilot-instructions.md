## opencode-docker

> OpenCode Docker — containerized environment for running OpenCode CLI.

# AGENTS.md

OpenCode Docker — containerized environment for running OpenCode CLI.

## Two usage modes

**`bin/opencode-docker`** (recommended for users) — uses `~/.opencode-docker/` for persistence, current dir as workspace.

**`make run`** (development only) — uses local `./homebase`, `./workspace`, `./secrets`. For working on this repo itself.

## Secrets

File-based, not env vars. Files in `/run/secrets` are loaded by `bootstrap.py` and converted to uppercase env vars:
- `anthropic_api_key` → `ANTHROPIC_API_KEY`
- Dashes and dots become underscores

Set up:
```bash
mkdir -p ~/.opencode-docker/secrets
echo "sk-..." > ~/.opencode-docker/secrets/anthropic_api_key
chmod 600 ~/.opencode-docker/secrets/*
```

## Distroless runtime constraints

Final image is `gcr.io/distroless/base-debian13`:
- **No `/bin/bash` or `/bin/sh`** — cannot `docker exec` into production container
- To debug: `make shell` (uses builder-tools stage with bash)
- Available commands: `mkdir find grep cat head tail sed awk echo ls cp mv rm chmod wc sort cut env pwd date dirname basename`
- Python 3, Node 24, git, Xvfb also available

## Build and run

```bash
make build                          # Build with auto-detected UID/GID
make build VERSION=1.3.17           # Build with version tag
make build-latest                   # Fetch latest opencode version, build and tag
make run                            # Dev run (uses local dirs)
make shell                          # Debug shell (builder-tools stage)
make clean                          # Remove image
```

Build requires `--build-arg USER_UID` and `USER_GID` matching the host user. Makefile auto-detects via `$(shell id -u)` / `$(shell id -g)`.

## Directory structure

| Path | Purpose | Gitignored? |
|------|---------|-------------|
| `config/` | opencode.json, custom skills | No (mounted rw in container) |
| `secrets/` | Local dev secrets | Yes |
| `homebase/` | Local dev home dir | Yes |
| `workspace/` | Local dev workspace | Yes |
| `superpowers/` | Local superpowers dir | Yes |
| `scripts/collect-runtime-deps.sh` | Collects binary+lib deps for distroless | No |
| `bootstrap.py` | Container entrypoint: secrets → Xvfb → opencode | No |

## Config

`config/opencode.json` defines MCP servers (context7, sequential-thinking) and loads the superpowers plugin. In the container this is mounted read-only at `/app/.config/opencode`.

Custom skills: `config/skills/<name>/SKILL.md`.

## Container security

Runs with `--read-only`, `--cap-drop=ALL`, `--security-opt=no-new-privileges`, 2GB memory, 2 CPUs. `/tmp` is tmpfs with exec permission (512MB).

## Brainstorming server

Superpowers brainstorming skill runs a local web server. Port is randomized and forwarded via `BRAINSTORM_PORT` env var. Access at `http://localhost:<port>`.

## Commit messages

Use conventional commits for persistent, searchable history:
- `feat:` new feature
- `fix:` bug fix
- `docs:` documentation only
- `chore:` maintenance, refactoring, build changes
- `refactor:` code change that neither fixes nor adds

Format: `type: short description` (under 72 chars). Example: `feat: add -w/--websearch flag to enable Exa web search`

---
> Source: [pkhamre/opencode-docker](https://github.com/pkhamre/opencode-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
