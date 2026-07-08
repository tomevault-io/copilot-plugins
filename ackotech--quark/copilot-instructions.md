## quark

> Monorepo framework for React component libraries. Provides CLI tooling, release automation, and dependency visualization.

# Quark

Monorepo framework for React component libraries. Provides CLI tooling, release automation, and dependency visualization.

## Project structure

```
quark/
├── cli/
│   ├── quark-cli/         # @quark-hq/quark — workspace scaffolding, package creation, Yalc workflows
│   ├── quark-scripts/     # @quark-hq/quark-scripts — release, dev publish, prod publish automation
│   └── quark-security/    # @quark-hq/quark-security — path, spawn, and validation helpers
├── atlas/                 # @quark-hq/atlas — Next.js dependency graph visualization app
├── packages/              # Component packages (scaffolded in consumer projects)
├── pnpm-workspace.yaml    # Workspace: packages/*, cli/*, app/*, atlas
└── quark-config.json      # Release and freeze configuration
```

## Tech stack

- TypeScript (CommonJS, target ES2020, strict mode)
- pnpm 9.x workspaces with Nx for task orchestration
- Jest for testing (ts-jest)
- Commander for CLI argument parsing
- Chalk 4.x (CJS-compatible) for terminal output

## Common commands

```bash
pnpm install                                          # Install all dependencies
pnpm run build:all                                    # Build all packages (nx run-many)
pnpm --filter @quark-hq/quark-security run build      # Build quark-security (must build first, others depend on it)
pnpm --filter @quark-hq/quark run coverage            # Test quark-cli
pnpm --filter @quark-hq/quark-scripts run coverage    # Test quark-scripts
pnpm --filter @quark-hq/quark-security run coverage   # Test quark-security
pnpm --filter @quark-hq/atlas run dev                 # Run Atlas dev server
```

## Build order

quark-security must be built before quark-cli or quark-scripts — they depend on it via `workspace:*`. The quark-cli `prebuild` script handles this automatically. When building manually, build quark-security first.

## Key conventions

- All spawn/exec calls must go through `spawnSyncSafe` from quark-security — never use `child_process` directly. This enforces path validation, arg sanitization, and `shell: false`.
- All file operations in scaffold/CLI code must use the safe wrappers from quark-security (`writeFileSyncSafe`, `readFileSyncSafe`, `mkdirSyncSafe`, etc.) which enforce path confinement.
- Package names are validated with `assertValidNpmPackageName` before use.
- Tests use Jest with `--watch` for local dev and `--coverage` for CI. Do not use `--watch` in CI.
- CLI version is read from `package.json` at runtime — do not hardcode version strings.

## Architecture notes

- **quark-cli** (`Init` class in `src/init/index.ts`) orchestrates the `quark new` flow by delegating to scaffold functions in `src/init/scaffold.ts` and environment checks in `src/init/environment.ts`.
- **quark-scripts** uses a hexagonal architecture: `app/` contains use cases, `ports/` defines interfaces, `infrastructure/` has implementations, `domain/` has business logic, and `cli/` wires everything via `CliContainer`.
- **quark-scripts** supports both Node (npm/pnpm) and Maven platforms via platform adapters. Each publish/release flow has its own adapter interface.
- Release flow: `release` creates a release map in `.release/map.json` → tag → `prod-publish` diffs tags and publishes changed packages.
- Prod publish checks `versionExistsInRegistry` before publishing to prevent duplicate versions.

## Platform support

macOS and Linux only. Windows is not supported.

---
> Source: [ackotech/quark](https://github.com/ackotech/quark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
