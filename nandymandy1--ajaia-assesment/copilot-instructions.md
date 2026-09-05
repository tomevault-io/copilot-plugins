## ajaia-assesment

> Before making code or structure changes, read and follow `STRUCTURE.md`. Treat it as the repo's structural source of truth.

# Agent Instructions

## First Rule

Before making code or structure changes, read and follow `STRUCTURE.md`. Treat it as the repo's structural source of truth.

## Scope

Work inside the existing app folders:

- `docs-lite-app`
- `docs-lite-server`

Do not scaffold replacement apps or rename those folders.

## Documentation

When changing conventions, architecture, test layout, config, aliases, or component structure, update the relevant docs in the same change:

- `STRUCTURE.md`
- `Progress.md`
- `AI_WORKFLOW.md`
- `HANDOFF.md`, when reviewer-facing context changes

## Verification

Run the checks that match the change.

Frontend:

```bash
cd docs-lite-app
npm run typecheck
npm run build
```

Backend:

```bash
cd docs-lite-server
npm run build
npx jest --watchman=false
```

Backend e2e:

```bash
cd docs-lite-server
npx jest --config ./test/jest-e2e.json --watchman=false
```

For documentation-only changes, builds/tests can be skipped, but mention that clearly.

---
> Source: [nandymandy1/ajaia-assesment](https://github.com/nandymandy1/ajaia-assesment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
