## aks-desktop

> Navigation guide for AI coding agents. For Headlamp plugin API examples see [`plugins/aks-desktop/AGENTS.md`](./plugins/aks-desktop/AGENTS.md).

# AGENTS.md

Navigation guide for AI coding agents. For Headlamp plugin API examples see [`plugins/aks-desktop/AGENTS.md`](./plugins/aks-desktop/AGENTS.md).

## Architecture Layers

| Layer | Path | Purpose | When to change |
| --- | --- | --- | --- |
| **Plugin** | `plugins/aks-desktop/` | AKS-specific UI, Azure integration, Kubernetes operations | Most changes go here |
| **Build** | `build/` | Plugin setup, external tool bundling (Azure CLI, Python), post-build verification | Packaging, bundled tool versions, installer behavior |
| **Headlamp fork** | `headlamp/` (submodule) | Electron shell, backend server, frontend framework | Only when plugin system cannot achieve the goal |

Headlamp fork commits must use a prefix:

- `aksd:` -- AKS Desktop-specific changes
- `upstreamable:` -- bug fixes, performance improvements, or features to contribute back upstream

See [`MAINTENANCE.md`](./MAINTENANCE.md) for the full fork rebase workflow.

## Decision Tree -- "Where does my change go?"

| Change type | Location |
| --- | --- |
| Azure CLI operations | `plugins/aks-desktop/src/utils/azure/<domain-module>.ts` (e.g., `az-acr.ts`, `az-identity.ts`, `aks.ts`) |
| Kubernetes operations | `plugins/aks-desktop/src/utils/kubernetes/` |
| GitHub integration | `plugins/aks-desktop/src/utils/github/` |
| New UI feature | `plugins/aks-desktop/src/components/<FeatureName>/` |
| Shared hooks | `plugins/aks-desktop/src/hooks/` |
| Shared types | `plugins/aks-desktop/src/types/` |
| Component-local types | Co-located in the component directory (e.g., `plugins/aks-desktop/src/components/DeployWizard/components/types.ts`) |
| Plugin registration | `plugins/aks-desktop/src/index.tsx` |
| Build / packaging | `build/` |
| Headlamp core | `headlamp/` (commit prefix: `aksd:` or `upstreamable:`) |

## Module Boundary Rules

> In the sections below, `src/` is shorthand for `plugins/aks-desktop/src/`.

- Feature components (`src/components/<Feature>/`) own their sub-components, hooks, utils, types, and tests.
- Cross-feature shared code lives in `src/utils/`, `src/hooks/`, or `src/types/`.
- Never import from another feature component's internals. If two features need the same logic, extract it to a shared location.
- No barrel files (enforced by ESLint `no-barrel-files` rule).

## Common Patterns

### Adding a wizard step

Wizard steps live in `src/components/<Wizard>/components/`. Each step is a standalone component (e.g., `BasicsStep.tsx`, `NetworkingStep.tsx`). See `DeployWizard` for the canonical example:

1. Create `<StepName>Step.tsx` in the wizard's `components/` directory.
2. Add the step to the wizard's step list in the parent wizard component.
3. Add tests (co-located `.test.tsx`) and optionally a `.stories.tsx`.

### Azure CLI command (`runAzCommand`)

New Azure CLI calls should go through `runAzCommand<T>()` in `src/utils/azure/az-cli-core.ts`, which provides structured error handling and typed responses. Domain-specific modules (e.g., `az-acr.ts`, `az-identity.ts`, `aks.ts`) wrap this function with typed helpers. Some older modules still call `runCommandAsync('az', ...)` directly. To add a new Azure operation:

1. Add a typed wrapper in the appropriate `src/utils/azure/<domain>.ts` module, or create a new domain module if none fits.
2. Call `runAzCommand<T>(args, debugLabel, errorContext, parseOutput)` and pass a `parseOutput` callback (for JSON output: `stdout => JSON.parse(stdout)`) -- it handles process execution, error wrapping, and returns `{ success, data, error }`.
3. Keep each azure util module single-responsibility.

### Route and sidebar entry

Register routes and sidebar entries in `src/index.tsx`:

1. Call `registerRoute()` with the path, component, and sidebar entry name.
2. Call `registerSidebarEntry()` for navigation.
3. Gate Azure-specific routes behind `Headlamp.isRunningAsApp()`.

### Project tab or overview section

Use `registerProjectDetailsTab()` or `registerProjectOverviewSection()` in `src/index.tsx`. Gate behind `isAksProject()` to show only for AKS-managed projects.

## Testing Conventions

- **Co-located tests**: `<Component>.test.tsx` alongside the component file.
- **Accessibility tests**: `<Component>.guidepup.test.tsx` files use guidepup for screen-reader-level a11y testing.
- **Fixtures**: `__fixtures__/` directories within feature components for shared test data.
- **Integration tests**: `src/utils/test/` for cross-module tests (Azure CLI, cluster settings, namespace utils).
- **Stories**: `.stories.tsx` files for Storybook visual testing, co-located with components.

## File Size Guidance

- Aim for under 400 lines per file. If a file exceeds this, look for opportunities to extract sub-components, hooks, or utility functions.
- Azure util modules (`src/utils/azure/`) should each cover a single domain (ACR, identity, federation, etc.). Split rather than grow a catch-all module.

## Cross-References

- **Headlamp plugin API examples and patterns**: [`plugins/aks-desktop/AGENTS.md`](./plugins/aks-desktop/AGENTS.md)
- **Fork maintenance and rebase workflow**: [`MAINTENANCE.md`](./MAINTENANCE.md)

---
> Source: [Azure/aks-desktop](https://github.com/Azure/aks-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
