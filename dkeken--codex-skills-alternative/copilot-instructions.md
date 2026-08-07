## codex-skills-alternative

> This repository is a library of **vendor-neutral Agent Skills** (`SKILL.md`) that reimplement the Codex *Creative Production* and *Product Design* plugin workflows. It is auto-loaded as agent context by Codex, Cursor, and GitHub Copilot.

# AGENTS.md

This repository is a library of **vendor-neutral Agent Skills** (`SKILL.md`) that reimplement the Codex *Creative Production* and *Product Design* plugin workflows. It is auto-loaded as agent context by Codex, Cursor, and GitHub Copilot.

## What lives here

- `skills/creative-*` — 8 image/creative-production skills + `creative-production` orchestrator
- `skills/design-*` — 9 product-design skills + `product-design` orchestrator
- `install.sh` / `uninstall.sh` — copy skills into each agent's skills directory

## Conventions for working in this repo

- Skills are **tool-agnostic**. Never hardcode a specific model/API/tool name inside a `SKILL.md`. Refer to "an image-generation tool", "a browser-automation tool", "a web-fetch tool".
- A skill `name` (frontmatter) MUST equal its folder name (lowercase, hyphens).
- Keep each `SKILL.md` short and focused — one workflow per skill.
- The two orchestrators route to sub-skills; they don't do the work themselves.
- See `CONTRIBUTING.md` before adding or editing skills.

## Using the skills (as an agent)

- Start creative work via `creative-production`; start design work via `product-design`. They route to the correct sub-skill.
- For Product Design, always begin with `design-get-context` unless `./design/context.md` already exists.
- For Creative Production, lock a visual direction (`creative-explore`) before generating final assets.
- Confirm with the user before any external deploy/tunnel (sends code off-machine).

---
> Source: [DKeken/codex-skills-alternative](https://github.com/DKeken/codex-skills-alternative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
