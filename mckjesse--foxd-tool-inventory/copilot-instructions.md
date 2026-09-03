## foxd-tool-inventory

> Orientation for any Claude Code session working in this repo. Read `README.md`

# CLAUDE.md — FOXD Tool Inventory

Orientation for any Claude Code session working in this repo. Read `README.md`
too — particularly the warning block at the top.

## What this repo is

Plant and equipment tracking across project sites: Django 5 + DRF backend
(`backend/`), React + Vite frontend (`frontend/`), Microsoft Entra ID auth.
`main` is the only branch that matters.

**This repo contains no deployment configuration** — no Dockerfile, no host
config. Do not conclude from that that the app is not deployed; hosting is
often configured in the host's dashboard instead, as it is for
`mckjesse/CRM-backend`. Check the host before changing anything that could
affect a running service.

There is **no test suite**, so a green CI run means the backend starts and the
frontend compiles — nothing about correctness. The README lists what was
actually verified.

## Non-negotiable invariant

**`AllocationHistory` is append-only and must record what actually happened.**
It is the whole point of this app over its predecessors — both earlier attempts
tracked where a tool *is* but not how it got there. Never update or delete a
history row to correct a mistake; write a new one. A tool move that does not
produce a history row is a bug, even if the tool's `project` field ends up
right.

## Conventions

- **Business logic in the viewset, thin serializers.** `views.py` is where
  allocation happens; keep it there rather than in model `save()` overrides,
  so the history write and the field update stay in one visible place.
- **Auth is Entra ID bearer tokens only** (`inventory/auth_backend.py`),
  validated against Microsoft's JWKS with audience and issuer checked. There is
  no local password login and there should not be one — the point of this
  choice is that it uses the Microsoft 365 tenant FOXD already runs.
- **`db.sqlite3` is per-developer and gitignored.** If you find yourself
  wanting to commit it, the real need is fixtures or a seed command.
- The frontend proxies `/api` to Django in dev (`vite.config.js`). Production
  needs `VITE_API_BASE` set to a real origin — which does not exist yet.

## CI

`.github/workflows/ci.yml` runs `manage.py check`, `makemigrations --check`,
`migrate`, and `vite build`. It deliberately does **not** run `manage.py test`,
because there are no tests — a green tick here means "it starts and it
compiles", nothing about correctness. Add the test step the moment tests exist.

## Where it came from

Extracted from `mckjesse/Claude` in August 2026 — one branch among fourteen
unrelated apps. The first five commits (`dde299c` → `bd28741`) are the
Variation Register, an unrelated app this branch line grew out of; its
`index.html` was removed here. This was the third of three attempts at a tool
inventory and was chosen for its data model. The others are archived in that
repo under `archive/2026-08-20/*`:

- `Tool-tracker-app` — zero unique commits, a strict subset of the next one.
- `claude/tool-inventory-app-A6PBz` — a vanilla-JS static app over a single
  Netlify Blobs key, with no allocation history at all. Worth revisiting only
  for its FOXD logo asset.

---
> Source: [mckjesse/foxd-tool-inventory](https://github.com/mckjesse/foxd-tool-inventory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
