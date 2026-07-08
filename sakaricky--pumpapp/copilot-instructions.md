## coding-standards

> Core JS/TS standards — ES modules, arrow functions, const/let, async/await


# Coding standards

- **Modules:** Use ES module `import` / `export` only. No `require()` or `module.exports`.

```typescript
// ❌ Avoid
require("dotenv").config();
const { PrismaClient } = require("@prisma/client");

// ✅ Prefer
import "dotenv/config";
import { PrismaClient } from "@prisma/client";
```

- **Functions:** Prefer arrow functions. For a single object argument, destructure in the parameter list.

```typescript
// ❌ Avoid
async function doSomething(options) {
  const { a, b } = options;
}

// ✅ Prefer
const doSomething = async ({ a, b, c }) => { };
const add = (x, y) => x + y;
```

- **Declarations:** Use `const` (or `let` when reassignment is needed). Avoid `var`.
- **Async:** Prefer `async/await` over `.then()` chains where it keeps code readable.

- **TypeScript:** Avoid `any`; use `unknown` or proper types instead.

- **Naming:** camelCase for variables and functions; PascalCase for types and interfaces. Use CONSTANT_CASE only for true constants (e.g. config keys, env names).

- **Exports:** Prefer named exports (`export const fn`, `export type T`) for utilities and types. Use default export only for pages or single-component entry files when the framework expects it.

- **Secrets:** No hardcoded secrets or credentials in the repo. Use environment variables and never commit `.env` with real values.

- **Database (dev):** After running `db:migrate` in development, always run `db:seed` so everyone has consistent baseline data. Require this in your workflow when setting up or resetting the local database.

---
> Source: [SakaRicky/PumpApp](https://github.com/SakaRicky/PumpApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
