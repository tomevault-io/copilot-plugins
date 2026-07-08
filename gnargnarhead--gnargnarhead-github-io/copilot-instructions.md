## gnargnarhead-github-io

> These rules are here to keep work **safe, predictable, and GitHub‑Pages friendly**.

# Agent Instructions (Website repo)

These rules are here to keep work **safe, predictable, and GitHub‑Pages friendly**.

## Safety Defaults (non‑negotiable)

- **Never start local servers** (no `python -m http.server`, `live-server`, `vite`, etc.). If a preview is needed, ask first.
- **No network installs/downloads** unless explicitly requested (no `npm install`, `bundle install`, `curl`, etc.).
- **No destructive commands** unless explicitly requested:
  - no `rm`, `git reset --hard`, `git clean -fd`, history rewrites, mass renames.
  - if an optimization requires replacing an asset, keep the original and propose the cleanup as a separate step.
- **No privilege escalation** (sandbox escape / sudo / system writes) unless the user explicitly approves.
- Prefer **offline verification**: link/path checks, basic HTML validation, and targeted build commands only when the repo actually has a build system.

## Repo Conventions

- This is a **static GitHub Pages site**. Prefer plain HTML/CSS; avoid adding JS or dependencies unless asked.
- Match the existing “90s cyber shrine” style:
  - keep table layout + panel classes
  - update nav links consistently across pages/layouts
- For file links in HTML:
  - if filenames contain spaces/commas, use **URL‑encoded paths** in `href`/`src`.

## Image / Asset Handling

- Optimize for GitHub storage:
  - strip metadata
  - prefer JPEG for photos/wallpapers (unless transparency is required)
- **Do not rename user assets** unless requested.
- When converting formats, keep both files until the user confirms which to keep.

---
> Source: [GnarGnarHead/GNARGNARHEAD.github.io](https://github.com/GnarGnarHead/GNARGNARHEAD.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
