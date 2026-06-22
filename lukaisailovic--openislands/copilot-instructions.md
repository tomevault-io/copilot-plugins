## openislands

> <!-- CLAUDE.md is the source of truth; AGENTS.md is a symlink to it. Keep it lean (~100 lines): invariants + commands + pointers. Deep reference lives in docs/. -->

<!-- CLAUDE.md is the source of truth; AGENTS.md is a symlink to it. Keep it lean (~100 lines): invariants + commands + pointers. Deep reference lives in docs/. -->

# OpenIslands — agent guide

Operating guide for working in this repo. Deep reference is linked, not inlined — read those
docs on demand instead of holding them in context.

## What it is

A local-first compiler + runtime for **agent-maintained data apps**. An app is a typed
**manifest** of reusable visual **islands** bound to **typed data contracts** built from local
files. You edit the manifest, never rendering code; `validate` fails loudly and names the island
when a binding and the data disagree. v1 is **live-only** — `serve` boots the TanStack Start SSR
runtime, queries files through the DuckDB core per request, and pushes SSE updates as files
change. No static export, no `generated/` snapshots.

## Repo map

```
packages/schema      # the contract — Zod → types + JSON Schema. The keystone; everything depends on it.
packages/storage     # swappable storage ports (ContentStore / AppStateStore / VersionStore) + local-disk adapters; configureStorage() swaps the backend
packages/compiler    # DuckDB query core: files → typed contracts; runs transforms/queries live
packages/runtime     # TanStack Start SSR app: island registry + React renderers
packages/cli         # the `openislands` command (init / validate / serve / add / infer)
packages/mcp-server  # the MCP edit loop (@openislands/mcp): read-many + patch_manifest/propose_edit + validate_sql
templates/           # empty (default), finance, health, operations — each ships .mcp.json + AGENTS.md + the skill; scaffolded by `init`
skills/openislands/  # the installable agent skill (npx skills add lukaisailovic/openislands --skill openislands); synced into every template
apps/examples/       # dogfood workspace (multi-app serve)
apps/docs/           # the docs site (Vocs); apps/docs/public/start.md is the raw agent-onboarding doc served at /start.md
```

## Invariants (don't break these)

- **`schema` is the keystone.** Change an island's config there and the CLI, runtime, and MCP
  follow. Adding an island touches three places — see `CONTRIBUTING.md`.
- **Never hand-edit build output** (`dist/`); the build owns it. There are no `generated/`
  snapshots — the runtime queries files live.
- **Server-side I/O goes through `@openislands/storage` ports, not raw `node:fs`.** App content →
  `getContentStore`, our `.openislands/` state → `getAppStateStore`, editor history →
  `getVersionStore`. Keep these stores keyed by the project dir (the opaque app key). The CLI's
  local scaffolding (init/add/infer) is exempt — it's local-only dev tooling.
- **The manifest stays declarative** — no transforms inside island configs; data shaping lives in
  the data/SQL layer.
- **Bind islands only to fields that exist; run `validate` after every manifest edit.** A missing
  binding is a named compile error — that's the safety net, don't work around it.
- **Prefer built-in islands.** Unknown types render a placeholder until a renderer exists under
  `components/custom/`.
- **Tests live in `test/`**, not `src/` (so the bundler doesn't ship them).
- **The agent skill has one source: `skills/openislands/SKILL.md`.** Edit it there, then `pnpm sync:skill`
  to regenerate the copies in `templates/*/.agents/skills/` and the shared `.mcp.json` + `AGENTS.md`
  (from `scripts/template-files/`); never hand-edit the synced copies. `validate:templates` re-syncs first.

## Commands

```bash
pnpm install && pnpm build
pnpm typecheck             # tsc --noEmit per package (the build strips types, never checks them)
pnpm test                  # vitest
pnpm lint                  # oxlint
pnpm validate:templates    # re-syncs the skill, then validates every template's manifest + bindings
pnpm sync:skill            # skills/openislands/ + scripts/template-files/ → templates/*/.agents, .mcp.json, AGENTS.md
node_modules/.bin/tsx packages/cli/src/index.ts serve templates/finance
```

Run `pnpm build && pnpm typecheck && pnpm test && pnpm lint` before calling anything done.
Toolchain: pnpm + Turborepo, tsdown (rolldown/Oxc) builds, oxlint + oxfmt, Vitest, ESM-only,
Node ≥ 20.

## Working style

- **UI uses Kumo UI** (`@cloudflare/kumo`, https://kumo-ui.com): use a Kumo component whenever one
  exists; write custom markup only when it doesn't.
- **Animating UI?** Use `/transitions-dev` for production-ready CSS transitions.
- **Reviewing UI?** Run `/web-interface-guidelines` against the changed components.
- **Verifying a served dashboard?** Drive it with `/agent-browser` rather than guessing.
- **Editing the docs site (`apps/docs`)?** It's built with Vocs — use the **vocs MCP**
  (`search_docs` / `read_page`) to look up Vocs components and config.
- **Before committing**, run `/simplify` on the changed code, then `pnpm build && pnpm typecheck && pnpm test && pnpm lint`.

## Deeper reference (read on demand)

- `docs/data-app-model.md` — manifest shape, island catalog + required fields, spans, filters,
  custom islands, workspaces.
- `docs/agent-edit-loop.md` — the MCP read-many/write-one loop, actions (data writes), queries (parameterized reads), connectors.
- `CONTRIBUTING.md` — adding an island, the PR gate, releases.
- `apps/docs/src/pages/` — the public docs (manifest, data contracts, islands, MCP).
- `packages/schema/src/index.ts` — the exact, authoritative island config schemas.

---
> Source: [lukaisailovic/openislands](https://github.com/lukaisailovic/openislands) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
