## mautic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

See `/CLAUDE.md` (monorepo root) for Druid.fi platform standards. This file documents this project's specifics only.

## What this repo is

Builds and publishes the `druidfi/mautic` and `druidfi/mautic-dxp` Docker images — Mautic marketing automation, patched with hotfixes and Druid-specific customizations, in a "plain" variant and a "DXP" variant (branded/wired for Drupal + Mautic integration, see `dxp/` projects in the monorepo root). It does not contain a running application of its own; it's a Dockerfile + build config + a small Mautic plugin, tested via `compose.yaml`.

Images are published to Docker Hub: [druidfi/mautic](https://hub.docker.com/r/druidfi/mautic/tags) and [druidfi/mautic-dxp](https://hub.docker.com/r/druidfi/mautic-dxp/tags).

## Architecture

**Multi-stage Dockerfile (`Dockerfile`)** layers on top of the official `mautic/mautic:<version>-apache` image (injected as the `mautic_upstream` build context, not a static `FROM`):

1. `base` — applies Composer hotfixes (specific `symfony/*` + `twig/twig` packages) and adds the `firemultimedia/mautic-multi-captcha-bundle` plugin. Also installs a few apt packages to fix upstream PHP errors, and copies in the custom Apache vhost, entrypoint script, and supervisord config.
2. `mautic_base_5` / `mautic_base_7` — copy the version-specific `DruidXPBundle` Mautic plugin from `files/5/plugins/` or `files/7/plugins/` respectively.
3. `mautic_dxp_5` / `mautic_dxp_7` — layer on DXP branding assets (favicon, logo, and for v5 also custom Twig template overrides from `files/5/app/`) from `files/shared/dxp/`.

**Two Mautic major versions are built in parallel** (currently 5.2.x and 7.1.x) because different customer projects pin different majors. `files/5/` and `files/7/` hold version-specific plugin code that has diverged (webhook command internals, composer.json) — when updating `DruidXPBundle`, check whether the change applies to one or both versions.

**`docker-bake.hcl`** defines the actual build matrix: 4 targets (`mautic-5`, `mautic-5-dxp`, `mautic-7`, `mautic-7-dxp`), each multi-arch (`linux/amd64`, `linux/arm64`), tagged with major, major.minor, and full version. The Mautic upstream version pin lives here (`contexts.mautic_upstream`), not in the Dockerfile — bump it here when updating Mautic core.

**`files/*/plugins/DruidXPBundle`** — a small custom Mautic plugin that adds a "Manage Content" menu item linking back to the paired Drupal site (only rendered when `DRUPAL_HOSTNAME` env var is set), plus CLI commands (`mautic:webhooks:create`, webhook update command) for managing webhooks from the console.

**`compose.yaml` / `.env`** — a local test harness only (not used in production deploys). Spins up the built image plus a MariaDB 10.11 container, wired for Traefik/Stonehenge routing at `MAUTIC_HOSTNAME`. The commented-out `mautic-cron` / `mautic-worker` services and volume mounts show the intended production shape (separate web/cron/worker roles sharing one image via `DOCKER_MAUTIC_ROLE`), but are disabled for local testing since the Makefile's `up` target only needs `mautic-web` + `mautic-db`.

**`files/entrypoint_mautic_web.sh`** — wraps the upstream Mautic entrypoint: optionally loads test fixtures, auto-installs Mautic if `MAUTIC_AUTO_INSTALL=true` and the DB is empty, runs pending Doctrine migrations on every boot if already installed, then execs into `apache2-foreground` or `php-fpm` depending on `FLAVOUR`.

**`files/supervisord.conf`** — runs Messenger queue consumers (`email`, `hit`, `failed` transports) as supervised background processes inside the same container; worker count and memory/time limits come from `DOCKER_MAUTIC_WORKERS_CONSUME_*`, `DOCKER_MAUTIC_WORKER_MEMORY_LIMIT`, `DOCKER_MAUTIC_WORKER_TIME_LIMIT` env vars.

**`composer.json`** at repo root is a metapackage manifest (not an installable app) — mostly documents the `mautic/core-lib` version constraint this repo targets.

## Common commands

```bash
# Build
docker buildx bake -f docker-bake.hcl --print   # print the build plan / resolved targets
make bake                                        # build all 4 targets locally (linux/arm64, no cache)
make bake-push                                   # multi-arch build + push to Docker Hub (needs Docker Hub creds; creates/uses a buildx builder)

# Local test environment (uses compose.yaml + .env)
make up                                          # start mautic-web + mautic-db, wait for healthy
make down                                        # tear down
make shell                                       # shell into the mautic-web container as www-data
make install                                     # run mautic:install inside the running container

# Console access
docker compose exec -it -u www-data -w /var/www/html mautic-web php ./bin/console --ansi <command>
```

`.env` controls which built image `compose.yaml` uses (`DOCKER_IMAGE`) and the local hostname (`MAUTIC_HOSTNAME`, routed via Stonehenge/Traefik at `*.docker.so`) — switch `DOCKER_IMAGE` between the `:5.x` and `:7.x` (and `-dxp` vs non-`-dxp`) tags to test a specific variant.

## Working in this repo

- When bumping the Mautic version, update the tag/version strings in `docker-bake.hcl` (both the `docker-image://` context and the `tags` list) — the Dockerfile itself has no version references.
- When adding a Composer hotfix or plugin, add it to the relevant `RUN composer update|require` block in the `base` stage of the Dockerfile so both major versions inherit it.
- `DruidXPBundle` code differs between `files/5/` and `files/7/` — verify changes against both directories, they're not symlinked or shared.
- DXP-only branding assets (logo, favicon, Twig overrides) live under `files/shared/dxp/` and `files/5/app/`; the non-DXP image build never touches them.

---
> Source: [druidfi/mautic](https://github.com/druidfi/mautic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
