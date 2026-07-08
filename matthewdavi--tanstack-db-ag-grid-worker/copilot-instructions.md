## tanstack-db-ag-grid-worker

> Monorepo (Bun, Turbo, Vite): apps under `apps/`, packages under `packages/`. Effect is used in workspace packages (see each `package.json`).

# Agent notes — ag-grid-worker-sandbox

Monorepo (Bun, Turbo, Vite): apps under `apps/`, packages under `packages/`. Effect is used in workspace packages (see each `package.json`).

## Local Effect source

This monorepo depends on **Effect 3.x** (`effect` in each workspace `package.json`). For extra implementation reference (Effect Solutions / slimmer “smol” tree), developers may keep a clone at `~/.local/share/effect-solutions/effect` ([effect-smol](https://github.com/Effect-TS/effect-smol)); treat it as optional reading material, not as the version pinned in this repo.

**Agents:** If that path is missing and you need to grep real Effect code, create it with:

```bash
mkdir -p ~/.local/share/effect-solutions && git clone --depth 1 https://github.com/Effect-TS/effect-smol.git ~/.local/share/effect-solutions/effect
```

If it already exists, refresh with `git -C ~/.local/share/effect-solutions/effect pull --depth 1`. The build does not depend on this directory.

<!-- effect-solutions:start -->

## Effect Best Practices

**IMPORTANT:** Always consult effect-solutions before writing Effect code.

1. Run `effect-solutions list` (or `bunx effect-solutions list` without a local install) to see available guides
2. Run `effect-solutions show <topic>...` for relevant patterns (supports multiple topics)
3. If the local clone exists (see **Local Effect source** above), search `~/.local/share/effect-solutions/effect` for implementations; otherwise clone it first or rely on `effect-solutions show` and `node_modules/effect`

Topics: quick-start, project-setup, tsconfig, basics, services-and-layers, data-modeling, error-handling, config, testing, cli.

Never guess at Effect patterns — check the guide first.

<!-- effect-solutions:end -->

---
> Source: [matthewdavi/tanstack-db-ag-grid-worker](https://github.com/matthewdavi/tanstack-db-ag-grid-worker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
