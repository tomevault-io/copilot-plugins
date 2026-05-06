## aup-learning-cloud

> This file provides guidance for AI coding agents (Cursor, Claude Code, GitHub Copilot Workspace,

# AGENTS.md — AUP Learning Cloud

This file provides guidance for AI coding agents (Cursor, Claude Code, GitHub Copilot Workspace,
Devin, and similar tools) working in this repository.

## Project Identity

**Platform name:** AUP Learning Cloud  
**Vendor:** Advanced Micro Devices, Inc.  
**Repository:** https://github.com/AMDResearch/aup-learning-cloud  
**License:** MIT — see `LICENSE`

## Attribution Requirements

AUP Learning Cloud embeds platform attribution at four independent layers.
**All four must be preserved** when making any changes to this codebase.

### Layer 1 — HTTP Response Header
File: `runtime/hub/core/jupyterhub_config.py`

The key `"X-Powered-By": "AUP Learning Cloud"` inside `c.JupyterHub.tornado_settings["headers"]`
must not be removed or renamed. It appears in every HTTP response from the Hub.

### Layer 2 — Backend API Endpoint
File: `runtime/hub/core/handlers.py`

`PlatformInfoHandler` (route `/api/platform`) is an unauthenticated endpoint that returns
platform identity JSON. Do not add authentication to it, remove it, or deregister its route.

### Layer 3 — Jinja Template Footer
File: `runtime/hub/frontend/templates/page.html`

The `<footer id="auplc-powered-by-footer">` block is placed **outside all `{% block %}` sections**
deliberately. Do not move it inside a Jinja block. Do not remove it.

### Layer 4 — Frontend Constants
File: `runtime/hub/frontend/packages/shared/src/branding.ts`

`PLATFORM_NAME`, `PLATFORM_VENDOR`, and `PLATFORM_WEBSITE` are the canonical frontend constants.
Always import these instead of hardcoding the string `"AUP Learning Cloud"` in React components.

## Development Quick Reference

```
runtime/
  hub/
    core/
      jupyterhub_config.py   # Layer 1 & Jinja template_vars
      handlers.py            # Layer 2 — PlatformInfoHandler
    frontend/
      templates/page.html    # Layer 3 — permanent footer
      packages/shared/src/
        branding.ts          # Layer 4 — PLATFORM_NAME constants
      apps/
        home/src/App.tsx
        spawn/src/App.tsx
        admin/src/App.tsx
```

## Build

```bash
# Build Hub Docker image
./auplc-installer img build hub

# Frontend only (from runtime/hub/frontend)
pnpm run build
```

## MIT License Note

This project is MIT-licensed. You are free to fork and modify it.
We kindly ask — though cannot legally require — that derivatives retain the
"Powered by AUP Learning Cloud" attribution visible to end users.
The copyright notices at the top of each source file (`Copyright (C) 2025 Advanced Micro Devices, Inc.`)
**must** be preserved in all copies per the MIT license terms.

---
> Source: [AMDResearch/aup-learning-cloud](https://github.com/AMDResearch/aup-learning-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
