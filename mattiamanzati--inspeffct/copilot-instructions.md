## inspeffct

> A CLI to help agents and humans inspect Effect applications by injecting observability into the environment or a debug connection.

# inspeffct

A CLI to help agents and humans inspect Effect applications by injecting observability into the environment or a debug connection.

## MANDATORY!

Before considering a task done:

- it should have tests for it
- there should be no lint errors
- the check commands should not output suggestions or warnings or errors
- pnpm test should succeed

## Status

TODOS.md reports the current status of things to do. Keep it updated.

## Context Repositories

The `.context/` folder contains reference repositories (gitignored). Clone them for implementation reference:

```bash
mkdir -p .context
git clone --depth 1 https://github.com/Effect-TS/effect.git .context/effect
git clone --depth 1 https://github.com/kentcdodds/cross-env.git .context/cross-env
git clone --depth 1 https://github.com/microsoft/vscode-js-debug.git .context/vscode-js-debug
```

- **effect**: The Effect library source code
- **cross-env**: Reference for spawning child processes with injected environment variables
- **vscode-js-debug**: Reference for NODE_OPTIONS injection, inspector URL detection, and CDP communication via bootloader pattern

## Project Structure

This is a pnpm monorepo with the following packages:

- `packages/inspeffct` - The main CLI package (published as `inspeffct`)

## Specs

See `SPECS.md` for the detailed architecture and implementation specs.

## Development

```bash
# Install dependencies
pnpm install


# Codegen (nefore building and testing and checking)
pnpm codegen

# Type check
pnpm check

# Build
pnpm build

# Run the CLI locally
node packages/inspeffct/dist/bin.js

# Lint
pnpm lint

# Run tests
pnpm test
```

## Package Manager

This project uses **pnpm** with workspace catalogs for dependency version management.

<!-- effect-solutions:start -->

## Effect Best Practices

**To grasp info about effect-related exports such as services/errors/layers** run `effect-language-service overview`

**Before implementing Effect features**, run `effect-solutions list` and read the relevant guide.

Topics include: services and layers, data modeling, error handling, configuration, testing, HTTP clients, CLIs, observability, and project structure.

**Effect Source Reference:** `./context/effect`
Search here for real implementations when docs aren't enough.

<!-- effect-solutions:end -->

---
> Source: [mattiamanzati/inspeffct](https://github.com/mattiamanzati/inspeffct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
