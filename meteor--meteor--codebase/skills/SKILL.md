---
name: codebase
description: Use when understanding the build system, modifying CLI commands, working with isobuild, or navigating the tools/ directory. Covers build pipeline flow and file locations.
---

# Codebase

Meteor's build system (Isobuild) and CLI structure.

## Overview

Meteor is a full-stack JavaScript platform with:
- **Core packages** in `/packages`
- **Build system (Isobuild)** in `/tools/isobuild`
- **CLI tool** in `/tools/cli`
- **Real-time data layer** via DDP
- **Mobile support** via Cordova

## Build Pipeline

1. **CLI** (`tools/cli/main.js`) → parses commands
2. **Project Context** (`project-context.js`) → resolves packages, dependencies
3. **Isobuild** (`tools/isobuild/`)
   - Bundler (`bundler.js`) → orchestrates build
   - Compiler (`compiler.js`) → compiles packages
   - Linker (`linker.js`) → wraps modules
   - Build plugins (Babel, TypeScript, CSS)
4. **Output** → `star.json`, programs
5. **Runners** (`tools/runners/`) → run-app.js, run-mongo.js, run-hmr.js
6. **Live App** → DDP Server ↔ Minimongo ↔ UI

## Directory Structure

```
tools/
├── cli/                   # Command-line interface
├── isobuild/              # Build system core
├── packaging/             # Package management
├── runners/               # App execution engines
├── fs/                    # File system utilities
├── cordova/               # Mobile/Cordova support
├── static-assets/         # Project templates
└── project-context.js     # Dependency resolution
```

## CLI (`tools/cli/`)

| File | Description |
|------|-------------|
| `main.js` | Entry point, command dispatcher |
| `commands.js` | Main command implementations |
| `commands-packages.js` | Package management commands |
| `commands-cordova.js` | Cordova/mobile commands |

**Commands:** `meteor create`, `run`, `build`, `deploy`, `add/remove`, `mongo`, `shell`

## Isobuild (`tools/isobuild/`)

| File | Description |
|------|-------------|
| `bundler.js` | High-level bundling orchestration |
| `compiler.js` | Package compilation |
| `linker.js` | Module wrapping and linking |
| `import-scanner.ts` | Import statement parsing |
| `compiler-plugin.js` | Compiler plugin API |
| `isopack.js` | Package format handling |

## Runners (`tools/runners/`)

| File | Description |
|------|-------------|
| `run-app.js` | Web application runner |
| `run-mongo.js` | MongoDB server runner |
| `run-hmr.js` | Hot module reload runner |
| `run-all.js` | Multi-runner orchestration |

## Build Targets

| Target | Description |
|--------|-------------|
| `web.browser` | Modern browsers |
| `web.browser.legacy` | Legacy browsers (IE11) |
| `web.cordova` | Cordova mobile apps |
| `server` | Node.js server |

## Package Relationships

- `tools-core` → rspack, future integrations
- `accounts-base` → all accounts-* packages
- `ddp-server` + `ddp-client` → realtime communication
- `mongo` → minimongo (client-side)
- `webapp` → all HTTP handling

## Project Templates

Via `meteor create --<template>`: `react`, `vue`, `svelte`, `angular`, `blaze`, `typescript`, `tailwind`, `solid`, `apollo`, `minimal`, `bare`, `full`

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `METEOR_PROFILE` | Build profiling |
| `METEOR_PACKAGE_DIRS` | Additional package paths |
| `METEOR_DEBUG_BUILD` | Verbose build output |

## Troubleshooting

- **Package not found:** Check `package.js` name, run `meteor reset`
- **Build plugin not running:** Check `archMatching`, file extensions
- **npm issues:** Clear `.meteor/local/`, run `meteor npm install`

---
> Converted and distributed by [TomeVault](https://tomevault.io) | [Claim this content](https://tomevault.io/claim/meteor/meteor)
<!-- tomevault:2.0:skill_md:2026-04-05 -->
