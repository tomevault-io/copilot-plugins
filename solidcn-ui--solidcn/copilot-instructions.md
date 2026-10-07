## registry-and-cli

> Open registry JSON, CLI schemas, and docs public/r parity


# Registry & CLI

## Schema source of truth

- `packages/cli/src/schema/json-schemas/registry.json`
- `packages/cli/src/schema/json-schemas/registry-item.json`
- `packages/cli/src/schema/json-schemas/config.json` (solidcn.json)

## Example hosted registry (docs site)

`apps/docs/public/r/` — `registry.json` and per-item `*.json` must stay **consistent**:

- Every entry in `registry.json` has a fetchable JSON file
- Paths follow CLI / `solidcn add` conventions

## After changing core components

1. Update or regenerate registry items if needed (`pnpm registry:build` from root when using that workflow)
2. Keep docs pages under `apps/docs/src/routes/docs/components/` and `apps/docs/src/lib/nav.ts` in sync

## CLI

- Do not hardcode absolute paths; use `cwd` / project config
- Registry commands (build, create, add) must align with the schemas above

---
> Source: [solidcn-ui/solidcn](https://github.com/solidcn-ui/solidcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
