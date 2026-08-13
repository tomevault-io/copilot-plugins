## toolbox

> <!-- do not edit directly - see templates in toolbox/context/templates/AGENTS.md.j2 -->

<!-- do not edit directly - see templates in toolbox/context/templates/AGENTS.md.j2 -->
# Security: Secrets & Credentials
- If you need a secret, check 1password with the `op` CLI - see the 1password skill
- NEVER output passwords, API keys, tokens, secrets, or credentials of any kind in messages, tool calls, tool results, thinking traces, or any other output. This includes in log output, error messages, debug info, or any content that could be visible to users or persisted.
- When displaying or returning results that contain secrets, redact them (e.g., `sk-****...****`).
- When checking environment variables or config files for debugging, mask or truncate any secret values before displaying them.

# Documentation and work-logging
- See the document-work skill

# Scripting
- Prefer python or typescript/javascript over bash unless absolutely necessary or its a very simple script
- When appropriate, create a Justfile with commonly used commands and keep it up to date. This file must not become a dumping ground for every command and must be kept organized and focused.

# Programming Language Guidance

## Python
- Prefer `uv` over other tools
- Prefer PEP 723 inline metadata for dependencies in small scripts, and use a `pyproject.toml` file for more complex projects

## Javascript
- Prefer `bun` and typescript over other tools

<!-- BEGIN generated: env (per-host ~/AGENTS.env.md) -->
# Environment: homeserver host

You are running on Ankit's homeserver (the host, not a container). Docker, the host
filesystem, and `/projects` are directly available. Containers run under `docker
compose` from `/home/ankit/hroot/{devserver,homeserver}/docker-compose*.yml`.

Path index - look here first:
user root: /home/ankit/hroot
homeserver: /home/ankit/hroot/homeserver
devserver: /home/ankit/hroot/devserver
projects: /home/ankit/hroot/projects (or /projects - symlink)

<!-- END generated: env -->

<!-- BEGIN generated: index (~/hroot/allplace/wiki/INDEX.md) -->
---
date-created: 2026-07-09 01:38 PM
date-modified: 2026-07-14 05:10 PM
---
Index of devices, domains, paths
## Network
All devices on Tailscale, addressed via short names.

## Devices
- `desktop-linux` — homeserver + dev box; RTX 2070 SUPER (8 GB VRAM) — runs the audio models (speaches, audiocpp)
- `desktop-win` — Windows PC + ML model server; RTX 5080 (16 GB VRAM) — runs the LLMs (Unsloth/llama-server)
- `m2book` — personal MacBook (client)

## Hostnames
Domains point to Tailscale IPs, caddy reverse proxies
- `*.home.ankitson.com`, `*.dev.ankitson.com` → desktop-linux
- `*.win.ankitson.com` → desktop-win

## Services
See Caddyfiles and Docker compose files for active services.
- desktop-linux: `~/hroot/homeserver/`, `~/hroot/devserver/`
- desktop-win: Caddyfile in `~/Documents/docs-root/projects/code/win-models/`

## Paths

### desktop-linux
- `~/hroot` — real home directory
- `~/hroot/homeserver`, `~/hroot/devserver` — Docker services
- `/projects/devdocker/dotfiles` — chezmoi dotfiles (shared across all machines)
- `~/hroot/allplace` — personal Obsidian vault (PARA); the only place notes are written
- `~/hroot/allplace/wiki` — cross-project engineering knowledge base (process, learning, evals, this index)
- `~/hroot/cybernetics` — OpenClaw agent operating memory (SOUL/MEMORY/daily logs; agents manage it)
- `/projects` → `~/hroot/projects` — active dev projects

### desktop-win
- `~/Documents/docs-root/projects/code/` — active dev projects
- `~/Documents/docs-root/projects/code/win-models/` — ML models, Caddy server

### m2book
- `~/Documents/docs-root/projects/code/` — active dev projects (shared Synology drive w/ desktop-win)
<!-- END generated: index -->

# Knowledge & Task Routing
Where written things go:
1. About one repo's code → that repo's `docs/` (dies with the repo)
2. Everything else worth writing down → the `allplace` Obsidian vault: `~/hroot/allplace/wiki/` for durable engineering knowledge (process, learnings, evals, machine index), elsewhere in the vault only for personal/narrative notes. Agents write only under `wiki/` and `agents/` there.
3. To-dos → beads (`bd`) where a project has it — never markdown TODO checklists
4. Agent runtime memory (OpenClaw) → `~/hroot/cybernetics`; agents manage it, humans don't write there

---
> Source: [ankitson/toolbox](https://github.com/ankitson/toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
