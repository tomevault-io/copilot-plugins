## hamlet

> the agent's job is to plan and execute coding: not to start and kill servers.

# Claude Project Memory

## Critical Rules

### agent executes, user runs
the agent's job is to plan and execute coding: not to start and kill servers.
ask the user if you want a server killed or started.

### Generated Files - NEVER MODIFY
- **NEVER** modify any generated files
- Generated files are marked with comments like "Generated from..." or "Auto-generated"
- If something doesn't work with generated files, fix the generator or build process, not the output
- Common generated file patterns:
  - `.generated/` directories (clobberable glue code)
  - `BuildAmp/` directories (generated Elm modules)
  - Files with "generated" in path or comments
  - `Storage.elm`, `Database.elm`, API files, etc.

### Code Generation - NEVER HARDCODE BUSINESS MODELS
- **NEVER** put business domain models directly in codegen source files
- Models are defined in Elm type aliases in `models/` directories
- Generators parse Elm files using tree-sitter in `core/elm-parser-ts.js`
- Follow "Elm once" principle - business types defined only in Elm, generated everywhere else
- Tree-sitter enables parsing of union types (custom types) for enum dropdowns, CHECK constraints
- Test mocks can use realistic examples, but generation code must be dynamic

### Server Management
- I do NOT start, stop, or kill servers
- User handles all server management due to node version issues
- I only make code changes and provide guidance

## BuildAmp System Architecture

### Package Structure
- **BuildAmp CLI**: `packages/buildamp/` - standalone code generation tool
  - `bin/buildamp.js` - CLI entry point
  - `lib/cli.js` - Command parsing (commander)
  - `lib/orchestrator.js` - Generation coordination
  - `lib/generators/` - Generators (db, api, storage, kv, sse, elm, handlers, sql, union-types)
  - `core/elm-parser-ts.js` - Tree-sitter Elm parser (type aliases, union types, records)
- **Hamlet Server**: `packages/hamlet-server/` - Runtime framework
- **Vite Plugin**: `packages/vite-plugin-buildamp/` - Dev server integration

### CLI Commands
```bash
buildamp gen --src app/horatio/models --dest app/horatio  # Generate all code from Elm models
buildamp gen api --src ... --dest ...                      # Generate for API models only
buildamp gen db --src ... --dest ...                       # Generate for DB models only
buildamp status                                            # Check generation status vs source models
```

### Vite Monorepo Structure
- **Build Tool**: Vite handles bundling and workspace dependency resolution
- **Framework Packages**: Single source of truth in `/packages/hamlet-server/`
- **Template Projects**: Reference framework via workspace dependencies, not duplicated files
- **Code Generation**: Generators in `packages/buildamp/lib/generators/`
  - Legacy re-exports in `shared/generation/` for backward compatibility

### TEA Handler Pool System
- Implemented fresh instance pool to fix state corruption bug
- Pool uses utilization-based replacement instead of persistent handlers
- Located in `/packages/hamlet-server/middleware/elm-service.js`
- Replaces persistent HandlerInstance objects with TEAHandlerPool

### Directory Structure
- Web app: `/app/horatio/web/`
- Generated glue: `/app/horatio/web/src/.generated/` (Elm) and `/app/horatio/server/.generated/` (JS)
- Shared generated: `/app/horatio/models/.generated/` (Config.elm shared between web/server)
- Server handlers: `/app/horatio/server/src/Api/Handlers/`
- Database migrations: `/app/horatio/sql/migrations/`
- **Elm models** (source of truth):
  - `/app/horatio/models/Schema/` - Database models (type aliases with Interface.Schema types)
  - `/app/horatio/models/Api/` - API endpoints (Request, Response, ServerContext)
  - `/app/horatio/models/Kv/` - KV store models
  - `/app/horatio/models/Storage/` - Browser localStorage models
  - `/app/horatio/models/Sse/` - Server-sent event types
  - `/app/horatio/models/Events/` - Background event types
  - `/app/horatio/models/Config/` - Configuration types

### Comment Submission System
- Complete implementation with auto-generated UUIDs
- Uses TEA handler pool for isolation
- Database operations include automatic host injection
- SubmitCommentHandlerTEA.elm handles the business logic

## Recent Fixes
- Fixed Storage.elm to properly import from BuildAmp.Storage.GuestSession
- Added vite-plugin-elm for proper Elm compilation in dev server
- Implemented auto-generated UUID migration for seamless ID generation
- Renamed `.hamlet-gen/` to `.generated/` and `Generated/` to `BuildAmp/` for clearer provenance

## Admin UI
- Schema-driven admin interface at `app/horatio/admin/`
- Reads `schema.json` at runtime (no code generation needed)
- CRUD operations for all tables with type-appropriate inputs
- Secure admin API with HAMLET_PROJECT_KEY protection (via auth-resolver tier)
- Tenant-isolated interface
- Next enhancement: relationship-aware UI (FK dropdowns, hide join tables)

## Test Management
- **Centralized test runner**: `./run_all_tests.sh` (requires Node 20)
- **Test suites**: 6 suites, ~412 tests total
  - Horatio Server: TEA handler pools (3 tests)
  - Hamlet Server: core, middleware, security (61 tests)
  - Create-BuildAmp: CLI parsing, codegen (17 tests)
  - BuildAmp Package: CLI, orchestrator, generators (331 tests)
  - BuildAmp Integration: file generation checks
  - Golden Model Verification: reference app + admin UI
- **BuildAmp tests**: `packages/buildamp/tests/` - includes elm-parser-ts, union-types, sql-generator, golden snapshots

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/justinTNT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
