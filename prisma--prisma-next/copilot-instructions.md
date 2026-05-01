## multi-plane-packages

> Guidelines for packages that span multiple planes


# Multi-Plane Packages

## Overview

Some packages need to span multiple planes (shared, migration, runtime) to provide both CLI entry points and runtime implementations. These packages use a structured directory layout to separate code by plane.

## Structure

Multi-plane packages use the following structure:

```
packages/{domain}/{package-name}/
├── src/
│   ├── core/              # Shared plane code
│   │   ├── adapter.ts    # Core implementation
│   │   ├── codecs.ts      # Codec definitions
│   │   └── types.ts       # Type definitions
│   └── exports/           # Entry points
│       ├── control.ts     # Migration plane (control plane descriptors)
│       ├── runtime.ts     # Runtime plane (runtime factories)
│       ├── adapter.ts     # Re-export from core
│       ├── codec-types.ts # Re-export from core
│       └── types.ts       # Re-export from core
```

## Architecture Config

Multi-plane packages require **multiple globs** in `architecture.config.json`, one for each plane:

```json
{
  "glob": "packages/targets/postgres-adapter/src/core/**",
  "domain": "targets",
  "layer": "adapters",
  "plane": "shared"
},
{
  "glob": "packages/targets/postgres-adapter/src/exports/control.ts",
  "domain": "targets",
  "layer": "adapters",
  "plane": "migration"
},
{
  "glob": "packages/targets/postgres-adapter/src/exports/runtime.ts",
  "domain": "targets",
  "layer": "adapters",
  "plane": "runtime"
}
```

## Plane Separation

- **`src/core/**`**: Shared plane code that can be imported by both migration and runtime planes
  - Core implementation (e.g., adapter, codecs, types)
  - No CLI-specific or runtime-specific dependencies
  - Can import from shared plane packages only

- **`src/exports/control.ts`**: Migration plane entry point (control plane)
  - Exports control plane descriptors (`ControlAdapterDescriptor`, `ControlTargetDescriptor`, `ControlDriverDescriptor`, etc.)
  - Can import from migration plane packages (e.g., `@prisma-next/cli`, `@prisma-next/core-control-plane`)
  - Can import from core (`../core/...`)
  - **Naming convention**: Use `control.ts` (not `cli.ts`) and export as `./control` (not `./cli`)

- **`src/exports/runtime.ts`**: Runtime plane entry point
  - Exports runtime factories (e.g., `createPostgresAdapter`)
  - Can import from runtime plane packages
  - Can import from core (`../core/...`)

## Build Configuration

All entry points can be built in a single `tsdown` configuration block:

```typescript
import { defineConfig } from '@prisma-next/tsdown';

export default defineConfig({
  entry: [
    'src/exports/adapter.ts',
    'src/exports/types.ts',
    'src/exports/codec-types.ts',
    'src/exports/control.ts',
    'src/exports/runtime.ts',
  ],
});
```

## When to Use Multi-Plane Structure

Use this structure when:
- ✅ Package needs both CLI entry points (for config files) and runtime code
- ✅ Package has shared code that both CLI and runtime need
- ✅ Package needs to import from both migration and runtime planes

**Examples:**
- Adapter packages (CLI descriptor + runtime adapter implementation)
- Target packages (CLI descriptor + runtime target-specific code)

## Benefits

- **Clear separation**: Plane boundaries are explicit in the directory structure
- **Dependency validation**: Dependency Cruiser can validate plane boundaries correctly
- **Flexibility**: Same package can provide both CLI and runtime functionality
- **Shared code**: Core implementation can be reused by both planes

## Related Patterns

- `.cursor/rules/shared-plane-packages.mdc`: Packages that are entirely in the shared plane
- `.cursor/rules/import-validation.mdc`: Import validation rules for plane boundaries

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
