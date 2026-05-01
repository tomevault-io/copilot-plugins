## import-rules

> Import rules for cross-package and self-import prevention


# Import Rules

## Cross-Package Imports

Do not use relative paths when importing across packages. Always use package imports instead.

- ❌ `import { something } from "../../../packages/database"`
- ✅ `import { something } from "@packages/database"`

## Self-Imports

Do not import from a package within itself.

- ❌ `import { Button } from "@packages/ui"` inside `packages/ui/`
- ✅ Use relative imports within the same package

---
> Source: [RhysSullivan/fastergh](https://github.com/RhysSullivan/fastergh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
