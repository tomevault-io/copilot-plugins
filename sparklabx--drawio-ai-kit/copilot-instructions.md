## drawio-ai-kit

> `drawio-ai-kit` helps an AI draw correct, beautiful draw.io diagrams.

# Repository Guidelines

`drawio-ai-kit` helps an AI draw correct, beautiful draw.io diagrams.

> [!IMPORTANT]
> **Hard rule (see `SKILL.md`)**: The kit is **read-only infrastructure**. Write generated `.drawio`/`.xml` output into the user's cwd, never into the repo folder.

## Tech Stack & Runtime
- **Node.js ≥18** (ESM, `.nvmrc` = 22) - `"type": "module"`; zero default exports (named exports only).
- **npm** (respect `package-lock.json`). Node built-in `node:test` (no test deps).
- For complete details on the architecture, directories, and files, see [docs/developer-guide.md](file:///Users/vanhungdo/Documents/Work/VCB/test/drawio-ai-kit/docs/developer-guide.md).

## Code Conventions & Common Patterns
- **No bundler, no transpiler, no TypeScript.** Plain `.mjs` only.
- **Catalog injection**: `loadCatalog()` returns the merged catalog; every `core.mjs` function takes `catalog` as first arg; `builder.mjs` stores it as `this.c`.
- **Declarative layout > coordinates**: Build node trees with `layout-engine.mjs` factories (`group`/`frame`/`grid` + `icon`/`box`) → `renderTree(d, root)` → `Diagram`. Hardcoding x/y coordinates violates the design pattern.
- **Color = Identity**: Never recolor AWS icons away from their category color (`colorFor`: entry.color → `categoryColors[category]` → `#232F3E`).
- **Nesting Hierarchy**: Group nesting order is enforced by `GROUP_LEVEL`: Cloud/Account/Region=0 → VPC=2 → AZ=3 → Subnet=4 → SG=5.
- **Edge Rounding Policy**: Tree/fanout roles → sharp (`rounded=0`); flow → type's `edgeCorner`.

---
> Source: [sparklabx/drawio-ai-kit](https://github.com/sparklabx/drawio-ai-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
