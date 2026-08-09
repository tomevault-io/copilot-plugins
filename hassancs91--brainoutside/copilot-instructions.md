## brainoutside

> Self-hosted memory server. A git repo of markdown is the user's brain;

# BrainOutside — working notes for agents

Self-hosted memory server. A git repo of markdown is the user's brain;
this Django app indexes it, serves it over REST + MCP with visibility
tiers enforced server-side, and gates every write behind human approval.

Single-user, single-brain. That is the product, not a limitation.

Full architecture and milestones: `docs/PLAN.md`. Deploying:
`docs/DEPLOY.md`. (Internal working docs — launch plan, release
checklist, design docs — live untracked under `docs/`, maintainer's
machine only; see `.gitignore`.)

**This repo is NOT the brain.** The brain is a separate repo the server
clones into `data/brain-repo`. Never edit brain content from here.

---

## Stack

Django 5.2 / Python 3.13, Postgres, Redis, Django-Q2. Docker Compose:
`web` + `mcp` + `worker` + `postgres` + `redis`.

| Path | What |
|---|---|
| `apps/mind/` | The public read API — endpoints, tier enforcement, file serving, throttling |
| `apps/brain/` | Entity index, git clone/sync, snapshots, graph |
| `apps/feeds/` | Proposal → approval queue → signed commit |
| `apps/reader/` | Reader agent, chat, SDK runner |
| `apps/brainconfig/` | Ops UI, settings, `/setup` wizard, health |
| `apps/events/` | Event log, SDK ledger, tasks |
| `apps/mcp_proxy/`, `apps/api_keys/` | MCP transport, consumer keys + tiers |
| `apps/docs/` | Self-documenting API reference at `/docs/` |
| `templates/ops/`, `templates/setup/` | The UI that matters |
| `assets/css/app.css` | Tailwind SOURCE (not served) |
| `static/css/tw.css` | Compiled artifact (committed, served) |

Settings: `base` → `dev` (DEBUG, SQLite) → `docker` (local full stack) →
`prod`. Containers run `config.settings.docker` locally,
`config.settings.prod` deployed.

---

## Running it

```
.\dev.ps1 up | reload [svc] | logs [svc] | status | manage <cmd> | css [-Watch]
```

- `web` self-reloads on Python edits. `mcp` and `worker` need `reload`.
- **Template edits need `reload web`.**
- The clone at `data/brain-repo` has no git credentials in-container:
  pull on the host, then `.\dev.ps1 manage sync_brain --no-pull`.

## CSS — read this before touching a template

Tailwind v4 via the standalone CLI. No Node, no `package.json`, no
`node_modules`. `static/css/tw.css` is a **committed build artifact**;
Docker does not build CSS.

**Adding a utility class to a template does nothing until you run
`.\dev.ps1 css` and commit the result.** A test enforces this.

- Edit `assets/css/app.css` (theme, components), not `tw.css`.
- `static/css/tokens.css` is the re-theming surface: colour variables
  only, loaded unlayered so it needs no rebuild. It has a `.dark` block —
  dark mode is a variable swap, so components need no `dark:` variants.
- Colour tokens live in `@theme inline`. This is load-bearing: a plain
  `@theme` resolves `var()` at `:root`, so a `.dark` subtree would
  inherit the already-resolved light value.
- Style against semantic tokens (`bg-surface`, `border-line`,
  `text-muted`), never a literal hex or a palette primitive.
- Status colours have two forms: `bg-signal`/`bg-danger` for **fills**,
  `text-signal-ink`/`text-danger-ink` for **text**. The fills fail
  contrast as text.
- `/ops/styleguide/` (DEBUG-only) renders every component in both themes.

## Tests

```
$env:DJANGO_SETTINGS_MODULE='config.settings.dev'
.\.venv\Scripts\python.exe -m pytest apps/core/tests apps/core/mcp/tests -q
```

Host venv, **not** the container — the container has no pytest, and the
host has no `django_redis`, so anything marked `django_db` cannot set up
a cache backend there. Keep new tests DB-free where possible.

## Guardrails — each exists because the mistake was made

`apps/core/tests/test_template_hygiene.py`, `test_css_build.py`:

| Fails on | Because |
|---|---|
| Inline `style="…"` in a template | Enforced CSP drops every one; a nonce cannot authorise attributes |
| Inline `<script>` without a nonce | CSP refuses it and the page still renders — the feature is just dead |
| Multi-line `{# … #}` | Django's hash comment is single-line; it renders verbatim into the page |
| Two `class` attributes on one tag | Browser keeps the first, silently discards the second |
| Tailwind directives in `static/*.css` | `collectstatic` post-processes it and an unresolvable `@import` fails the deploy |
| `tw.css` differing from a fresh build | The artifact went stale; classes compile to nothing |

CSP is **enforced in dev** on purpose. Report-only hid a release blocker
for the life of the project. Do not turn it off.

---

## Gotchas that cost real time

- **PowerShell has no heredoc.** `git commit -m @'…'@` mangles the
  message into pathspecs. Write to a file, use `git commit -F`.
- **Don't pipe native commands through `2>&1` in PowerShell** — raises
  `NativeCommandError` on success. Docker's normal output goes to stderr.
- **Playwright: `wait_until="domcontentloaded"` on ops pages**, never
  `networkidle` — `activity.json` polls every 4s so they are never idle.
- **Alpine, two separate DOM-mutation-during-submit bugs:** never disable
  a submit button synchronously in `@submit` (a disabled submitter is
  excluded from form data, so `action` vanishes), and never mutate state
  a `<template x-if>` is keyed on (it detaches the form and the POST
  never fires). Defer with `setTimeout`.
- **Alpine `:style="'a:' + b"` (string) is dropped by CSP** — it compiles
  to `setAttribute`. The object form `:style="{a: b}"` writes through the
  CSSOM and is fine. Prefer `:class`.
- **`docker exec` runs as root; the app user is `app`** — use
  `docker exec -u app` for any git operation in a container.
- **Git Bash mangles `/`-leading args** into Windows paths. Use
  `MSYS_NO_PATHCONV=1`, or run from PowerShell.
- Windows bind mounts occasionally throw `PermissionError` on the
  snapshot rmtree/rename. Transient, but the non-atomic swap is a real
  logged issue.

## Conventions

- **Verify live against the running stack before claiming it works.** For
  UI that means a real browser: Playwright is installed globally at
  `C:\Python313` (not in `.venv`). Log in by minting a session cookie
  from a Django shell, then assert against the rendered DOM. Prefer an
  independent recomputation of what the UI claims over checking that it
  rendered.
- Prod-only code paths need a prod-settings check. `dev` uses plain
  static storage and would not have caught the `collectstatic` failure
  that crash-looped deploys.
- Commit each step separately. Say what was verified **and what was not**.
- **"Is this engine, or user-config?"** User-config goes in
  settings/DB/brain-repo, never in code. Engine code must grep clean for
  personal values.
- Any button triggering background work must disable while running and
  appear on `/ops/tasks/`.

---
> Source: [hassancs91/brainoutside](https://github.com/hassancs91/brainoutside) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
