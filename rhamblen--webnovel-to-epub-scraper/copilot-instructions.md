## webnovel-to-epub-scraper

> > Repo-specific conventions for this project. Universal rules (e.g. how to

# Project instructions — phased build/automation repo

> Repo-specific conventions for this project. Universal rules (e.g. how to
> publish a GitHub release) live in the global `~/.claude/CLAUDE.md`.

## Documentation conventions — phased project repos

For a build/automation project meant to be published or returned to later,
structure it so a future reader understands *what we did and why*.
Default layout:

- **README.md** — lead with a one-line "what + why"; then what-it-does (by phase/feature),
  stack, repo layout, status, license. Link into `docs/`.
- **docs/project-plan.md** — phased roadmap. Each phase: **Objective · What we build ·
  Prerequisites · Deliverables · Why (rationale) · Exit criteria**. A live **status table**
  at the top (☐ not started · ◐ in progress · ☑ done), plus an **Open decisions** section.
- **docs/architecture.md** — design principles, key trade-offs, hardware/topology, data sources.
- **docs/ai-context.md** — cold-start orientation map for the *next AI session*: repo purpose,
  how-to-work rules, environment/IDs, key entities, build-phase table, gotchas. Dense, factual,
  link-rich, not for end users. Required in every repo so a cold session can work safely.
- **docs/decisions/NNNN-title.md** — ADRs for each significant choice: *Status, Date, Context,
  Decision, Consequences*. This is the durable "why".
- **docs/inventory.md** (or baseline) — the starting-state facts the build assumes.
- **docs/phases/** — one build-log per phase: what was built (with IDs), how to reproduce,
  decisions/surprises, verification.
- **CHANGELOG.md** — Keep a Changelog + SemVer; update **every phase**; `[Unreleased]` accumulates.
- **LICENSE** (MIT default) and **.gitignore** (never commit secrets/credentials/runtime state).

Principle: the **repo is the source of truth for intent + rationale**; version-controlled
config goes in the repo as code (YAML/etc.), never hand-edit opaque state stores. Phases map
loosely to minor versions (Phase 1 → v0.1.0). Releases only when explicitly asked.

## Versioning — the web UI shows the running build

`docker/app/__init__.py`'s `__version__` is the **single source of truth**. It titles the
FastAPI app and renders in the web UI footer, so a deployed container can be identified at
a glance — which only helps if it's true. `tests/test_version.py` enforces that, so the
version can't quietly drift from the CHANGELOG.

- **While developing:** `__version__` carries a `-dev` suffix (e.g. `0.7.0-dev`), meaning
  unreleased work on top of the last tag. The footer shows it, so a dev build is never
  mistaken for a release. Work accumulates under CHANGELOG `[Unreleased]`.
- **When releasing vX.Y.Z** (in addition to the global release steps), *before* tagging:
  1. Set `__version__ = "X.Y.Z"` (drop `-dev`).
  2. Promote CHANGELOG `## [Unreleased]` → `## [X.Y.Z] — <date>`.
  3. Run `pytest tests/test_version.py` — it fails unless the app version is the newest
     released CHANGELOG section, which is what makes the footer match the GitHub Release.
  4. After tagging, bump `__version__` to the next `-dev` (e.g. `0.8.0-dev`).

The tag is `vX.Y.Z`; `__version__` has no `v` prefix (the footer adds it for display).

## Deploying — images are built in CI, not on the server (ADR 0011)

Pushing to `main` builds and publishes both images to GHCR; the server pulls them. **An
ordinary change needs a push, not a Release** — tags are only for versions you mean:

| you do | CI publishes |
|---|---|
| push to `main` | `:latest`, `:sha-<short>` |
| push a `v*` tag | `:X.Y.Z`, `:X.Y`, `:latest` |

Tests gate the publish, so a red suite never becomes the deployed image. Consequence to
remember: **a local edit is not deployable until it's pushed**; there is no longer a
copy-to-server shortcut.

The server holds **only** `deploy/docker-compose.yml` + `deploy/.env` — never application
source — and updates via Compose Manager → *Update Stack*. Both sit **with the data in one
folder** (superseding ADR 0005's split, see ADR 0011):

```
/mnt/user/appdata/webnovel-to-epub-scraper/
├── docker-compose.yml
├── .env
└── config/app.db          ← bind-mounted to /config
```

There is no `...-scraper-docker/` folder any more. The old pair of names one suffix apart
is what caused the 2026-07-04 database deletion, and nothing is copied into appdata now,
so the separation had no remaining purpose.

The footer shows the version plus the commit the image was built from
(`v0.7.0-dev (a1b2c3d)`), because with `:latest` and no source on the server, that stamp is
the only reliable answer to "what's actually running?".

---
> Source: [rhamblen/webnovel-to-epub-scraper](https://github.com/rhamblen/webnovel-to-epub-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
