## petri

> Petri is a workflow generator for lab automation. It provides a visual node-based editor for

# CLAUDE.md

Petri is a workflow generator for lab automation. It provides a visual node-based editor for
creating laboratory protocols that execute on automation platforms like Opentrons. The system
compiles high-level workflow definitions into specific robotic actions (pipetting, mixing,
thermocycling, etc.).

## Build System: Bazel

This project uses Bazel as its primary build system with a polyglot stack (TypeScript, Rust,
Protocol Buffers).

### Common Commands

**Build the application:**
```sh
bazelisk build //petri:runner
```

**Run the server locally:**
```sh
bazelisk build //petri:runner && \
    BAZEL_BINDIR='bazel-bin' sisyphus app run-config \
    --binary bazel-bin/petri/runner_/runner \
    --config petri/server.star \
    --environment dev
```

**Run all tests:**
```sh
bazelisk test //petri/client/...
```

**Run a specific test file:**
```sh
bazelisk test //petri/client/nodes:thermocycling_node_test
```

**Run TypeScript type checking:**
TypeScript checking is integrated into the build. Use the tsconfig at the root which extends `gts` (Google TypeScript Style).

## Architecture

### High-Level Structure

The codebase is divided into three main layers:

1. **Server (`petri/server.ts`)**: Fastify-based HTTP server that serves the web UI and provides Connect RPC services. Handles SSR (server-side rendering) of the React-like UI using the Corgi framework.

2. **Client (`petri/client/`)**: Browser-based visual editor built with Corgi (a custom React-like framework). Contains:
   - **Node system** (`nodes/`): Individual node types (mix, thermocycle, serial dilutions, etc.)
   - **Execution engine** (`execution/`): Graph evaluation, action dispatching, and robot-specific handlers (Opentrons)
   - **UI controllers**: Graph editor, table editor, route controller
   - **Type system** (`types/`): Type definitions for materials, actions, units, lab equipment

3. **Logic (`petri/logic/`)**: Rust/WASM module compiled with wasm-bindgen for browser execution. Currently handles Starlark evaluation.

### Key Architectural Patterns

**Graph-Based Execution Model**:
- Workflows are represented as directed acyclic graphs (DAGs) of nodes
- Each node has typed inputs/outputs (materials, plates, volumes, etc.)
- The `Graph` class (`execution/graph.ts`) manages nodes, links, and constraints
- Execution happens via the "pipes" system (`execution/pipes.ts`) which propagates data through the graph

**Node Specifications**:
- All nodes are defined in `petri/client/nodes/`
- Each node exports a `NODE` constant satisfying the `NodeSpec` interface
- `NODE_SPECS` array in `nodes/nodes.ts` is the registry of all available nodes
- Nodes define:
  - `inputs`/`outputs`: Pad specifications with types
  - `pipe()`: Function that wires up data flow in the execution overlay

**Action Dispatching**:
- Abstract actions (pipette, mix, thermocycle) are defined in `types/actions.ts`
- Platform-specific handlers translate actions to robot commands
- `OpentronActionsHandler` (`execution/opentrons_actions_handler.ts`) converts to Opentrons protocol steps
- Handlers can be layered: logging, profiling, simulation

**Type System**:
- Strong typing for lab concepts: `MaterialValue`, `PlatedMaterialValue`, `PlateValue`, `VolumeValue`
- Types include units with automatic conversion (see `types/units.ts`)
- Constraints system ensures type safety across node connections

### UI Framework: Corgi

This project uses Corgi, a custom React-like framework from `@dev_april_corgi`:
- JSX with custom factory: `corgi.createVirtualElement`
- Component binding via `corgi.bind()` with controller pattern
- SSR support with hydration via `corgi.hydrateElement()`
- Controllers manage state and handle events

### Build Configuration

**Bazel workspace structure**:
- `MODULE.bazel`: Defines Bazel module dependencies (rules_js, rules_ts, rules_rust, etc.)
- `BUILD.bazel` files define build targets in each directory
- Uses Aspect Build's rules for JS/TS (`@aspect_rules_js`, `@aspect_rules_ts`)
- Rust compilation via `@rules_rust` with WASM target support

**Dependencies**:
- npm packages managed via pnpm (`pnpm-lock.yaml`)
- Bazel rule `npm_translate_lock` converts pnpm lock to Bazel targets
- Rust crates managed via Cargo with `crate_universe` extension

**TypeScript compilation**:
- Custom rule `c_ts_project` from `@dev_april_corgi` (wrapper around ts_project)
- Bundling via `esbuild_binary` for both client and server
- Supports source maps for debugging

### Data Flow

1. User builds workflow graph in browser UI
2. Graph is serialized to JSON (see `execution/json.ts`)
3. On execution, graph is converted to "piped overlay" representation
4. Nodes' `pipe()` functions establish data sources and transformers
5. Sources are evaluated, triggering cascading computation
6. Actions are dispatched to appropriate handler (e.g., Opentrons)
7. Handler generates platform-specific protocol or executes directly

### Testing

Tests use Jest with jsdom environment for DOM testing:
- Test files: `*.test.ts` or `*.test.tsx`
- Most tests in `petri/client/nodes/` and `petri/client/execution/`
- Run individual tests via Bazel targets (e.g., `//petri/client/nodes:mixing_transformer_test`)

## External Dependencies

This project depends on two external Bazel modules:

1. **`@dev_april_corgi`**: Provides the Corgi UI framework and build rules (`c_ts_project`, `esbuild_binary`)
2. **`@dev_april_sisyphus`**: Provides deployment tooling (`sisyphus_pushable`, app runtime)

These are fetched via `archive_override` in `MODULE.bazel`.

---
> Source: [escalante-bio/petri](https://github.com/escalante-bio/petri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
