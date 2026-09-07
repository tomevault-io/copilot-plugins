## 032-import-rules

> Import rules for cross-package and self-import prevention


# Import Rules

## Cross-Package Imports

Do not use relative paths when importing across packages. Always use package imports instead.

- ❌ `import { something } from "../../../packages/core"`
- ✅ `import { something } from "@transcripts-mcp/core"`

## Self-Imports

Do not import from a package within itself using the package name.

- ❌ `import { name } from "@transcripts-mcp/core"` inside `packages/core/`
- ✅ Use relative imports within the same package

## Examples

```typescript
// ✅ Good: Cross-package import using package name
import { name as core } from "@transcripts-mcp/core";
import { name as adapters } from "@transcripts-mcp/adapters";

// ✅ Good: Relative import within same package
import { helper } from "./utils";
import { types } from "../types";

// ❌ Bad: Relative path across packages
import { name } from "../../core/src";
```

---
> Source: [Stormix/transcripts-mcp](https://github.com/Stormix/transcripts-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
