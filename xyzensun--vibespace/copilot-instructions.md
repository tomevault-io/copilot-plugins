## vibespace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vibe Space is a web-based tool that generates containerized development environment configurations. Users configure their stack through a 6-step wizard UI, and the system produces five files: `Dockerfile`, `docker-compose.yml`, `entrypoint.sh`, `deploy.sh`, and `.cnb.yml`. There is no build step — the frontend is purely static HTML/JS served directly.

## Development

**No build system** — open `index.html` directly in a browser. All dependencies are loaded via CDN (Alpine.js, Tailwind CSS, Prism.js, JSZip).

**CI pipeline** (GitHub Actions, manual trigger only):
```bash
# Lint the test Dockerfile
docker run --rm -i hadolint/hadolint < build-test/Dockerfile
# Build the all-in-one test image
docker build -t diy-vibe-space:aio build-test/
# Validate compose config
docker compose -f build-test/docker-compose.yml config --quiet
```

The `build-test/` directory contains a pre-generated reference configuration with all features enabled, used by CI to verify the Docker build and tool installations.

## Architecture

### Data Flow

```
UI (index.html + Alpine.js) → appState (js/app.js) → generators → output preview
```

All state lives in a single `appState()` Alpine.js component. Every config property is watched via `$watch`, so any change triggers `generate()` which calls all five generators and refreshes syntax highlighting.

### Key Modules

- **`js/app.js`** — Alpine.js state: wizard navigation, toggle helpers, preset loading, config collection (`getConfig()`), generation orchestration
- **`js/data/urls.js`** — Centralized URL registry (`URLS`): all remote links (CDN, mirrors, language installers, tool downloads, ZCF templates) defined here. Template URLs use functions (e.g. `URLS.languages.nodejs.setup('20')`). Loaded before `defaults.js`.
- **`js/data/defaults.js`** — All configuration constants: languages (versions, apt packages, dev tools), AI tools, mirror sources (referencing `URLS`), MCP presets, Claude workflows/output styles, presets
- **`js/generators/dockerfile.js`** — Builds a multi-layer Dockerfile from config. Layers are ordered for cache efficiency: system packages → languages → npm globals → dev tools → code-server → AI tools → SSH
- **`js/generators/compose.js`** — Generates docker-compose YAML with port mappings, volumes, and environment variables
- **`js/generators/entrypoint.js`** — Generates the container startup script (git config, SSH keys, passwords, dynamic README, service launch)
- **`js/generators/deploy.js`** — Generates a user-friendly deployment helper script
- **`js/generators/cnb.js`** — Generates `.cnb.yml` for CNB cloud platform. CNB uses `env` (not `environment`) for env vars, and data persistence via symlinks to `/root/.cnb/` in `stages` (not `volumes`)
- **`js/utils/download.js`** — Single file download and ZIP packaging via JSZip
- **`js/utils/highlight.js`** — Prism.js syntax highlighting wrapper

### Generator Pattern

Each generator is a pure function: `generateX(config) → string`. The config object is assembled by `appState.getConfig()` from current UI state. Generators are independent and can be modified without affecting each other.

### Region-Aware Generation

The `region` field (`china` | `international`) controls mirror source injection throughout all generators — apt mirrors, npm registry, pip index, Go proxy, and GitHub proxy for downloads.

## Conventions

- Chinese comments for user-facing features, English for technical logic
- The project uses no package manager — no `package.json`, no `node_modules`
- Commit messages follow the pattern: `update:描述` or `fix:描述`
- Base image is fixed at Ubuntu 24.04 LTS (`DEFAULTS.baseImage`)

---
> Source: [XyzenSun/vibespace](https://github.com/XyzenSun/vibespace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
