## esm-imports

> This project uses ECMAScript Modules (ESM) exclusively. Follow these rules when writing imports:

# ESM Import Rules

This project uses ECMAScript Modules (ESM) exclusively. Follow these rules when writing imports:

## Always include .js extension

When importing local files, always include the `.js` extension in the import path.

```typescript
// ✅ Good
import { Database } from './interfaces/database.js';

// ❌ Bad
import { Database } from './interfaces/database';
```

## Handling CommonJS modules

Some dependencies are CommonJS modules. When importing them, use this pattern:

```typescript
// ✅ Good
import pkg from 'pg';
const { Pool } = pkg;

// ❌ Bad
import { Pool } from 'pg';
```

## Import ordering

Follow this order for imports:
1. Node.js built-in modules
2. External dependencies
3. Local imports

```typescript
// ✅ Good
import fs from 'fs';
import path from 'path';

import express from 'express';
import pkg from 'pg';
const { Pool } = pkg;

import { Database } from './interfaces/database.js';
import { SQLiteConfig } from './databases/sqlite.js';

// ❌ Bad (mixed order)
import express from 'express';
import { Database } from './interfaces/database.js';
import fs from 'fs';
import pkg from 'pg';
const { Pool } = pkg;
```

## Named exports

Prefer named exports over default exports for better IDE auto-import support:

```typescript
// ✅ Good
export class PostgresDatabase implements Database {
  // ...
}

// ❌ Bad
class PostgresDatabase implements Database {
  // ...
}
export default PostgresDatabase;
``` 

---
> Source: [cuongtl1992/mcp-dbs](https://github.com/cuongtl1992/mcp-dbs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
