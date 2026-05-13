## convex-structure

> Convex folder structure and function organization guidelines


# Convex Folder Structure Guidelines

## Overview

This project organizes Convex code by feature folders. Each feature lives in `convex/{feature}/` with an `index.ts` that declares Convex queries and mutations by wiring validators and handlers from files in `queries/` and `mutations/`.

## Core Structure Pattern

### 1. Main Feature Entry File

- **Location**: `convex/{feature}/index.ts`
- **Purpose**: Export the Convex runtime functions using `query` and `mutation`
- **Pattern**: Import `{FunctionName}Args` (when needed) and `{FunctionName}Handler` from the function file

**Example**: `convex/casinos/index.ts`

```typescript
import { mutation, query } from '../_generated/server';
import { createCasinoArgs, createCasinoHandler } from './mutations/createCasino';
import { createMultipleCasinosArgs, createMultipleCasinosHandler } from './mutations/createMultipleCasinos';
import { deleteCasinoArgs, deleteCasinoHandler } from './mutations/deleteCasino';
import { updateCasinoArgs, updateCasinoHandler } from './mutations/updateCasino';
import { getAllCasinosHandler } from './queries/getAllCasino';
import { getCasinoByIdArgs, getCasinoByIdHandler } from './queries/getCasinoById';
import { getCasinosByStateArgs, getCasinosByStateHandler } from './queries/getCasinosByState';

// Queries
export const getAllCasinos = query({ handler: getAllCasinosHandler });
export const getCasinoById = query({ args: getCasinoByIdArgs, handler: getCasinoByIdHandler });
export const getCasinosByState = query({ args: getCasinosByStateArgs, handler: getCasinosByStateHandler });

// Mutations
export const createCasino = mutation({ args: createCasinoArgs, handler: createCasinoHandler });
export const createMultipleCasinos = mutation({
  args: createMultipleCasinosArgs,
  handler: createMultipleCasinosHandler,
});
export const deleteCasino = mutation({ args: deleteCasinoArgs, handler: deleteCasinoHandler });
export const updateCasino = mutation({ args: updateCasinoArgs, handler: updateCasinoHandler });
```

### 2. Feature Folder Structure

- **Location**: `convex/{feature}/`
- **Contents**:

```
convex/{feature}/
├── index.ts           # Entry: wires handlers/args into Convex queries/mutations
├── {feature}.model.ts # (optional) model helpers/types for the feature
├── queries/           # Read operations (QueryCtx)
└── mutations/         # Write operations (MutationCtx)
```

### 3. Function File Pattern (queries/mutations)

Each function file exports:

- `{FunctionName}Args`: Plain object of Convex validators (omit if no args)
- `{FunctionName}Handler`: Async function implementing business logic
- `{FunctionName}Returns` (optional): Validator describing structured returns when useful

Signatures use Convex types from `convex/_generated/server` and `convex/values`.

Examples from current codebase:

```typescript
// Mutation example: convex/casinos/mutations/createCasino.ts
import { v } from 'convex/values';
import { MutationCtx } from '../../_generated/server';
import { Id } from '../../._generated/dataModel';

export const createCasinoArgs = {
  name: v.string(),
  website: v.optional(v.string()),
  license_status: v.optional(v.string()),
  source_url: v.optional(v.string()),
  state_id: v.id('states'),
  is_tracked: v.boolean(),
};

export const createCasinoHandler = async (
  ctx: MutationCtx,
  args: {
    name: string;
    website: string;
    license_status: string;
    source_url: string;
    state_id: Id<'states'>;
    is_tracked: boolean;
  }
) => {
  return await ctx.db.insert('casinos', { ...args });
};
```

```typescript
// Query example: convex/casinos/queries/getCasinoById.ts
import { Id } from '../../_generated/dataModel';
import { QueryCtx } from '../../_generated/server';
import { v } from 'convex/values';

export const getCasinoByIdArgs = { id: v.id('casinos') };

export const getCasinoByIdHandler = async (ctx: QueryCtx, args: { id: Id<'casinos'> }) => {
  return await ctx.db.get(args.id);
};
```

Notes:

- If a query has no inputs, omit `args` in both the file and the `query({ ... })` declaration (see `getAllCasino`).
- Define `{FunctionName}Returns` only when you need runtime validation of a complex response.

## Naming Conventions

- **Feature folder**: `convex/{feature}/`
- **Entry file**: `index.ts`
- **Operations**: `queries/{functionName}.ts`, `mutations/{functionName}.ts`
- **Exports per file**: `{functionName}Args` (optional), `{functionName}Handler`, `{functionName}Returns` (optional)

## Benefits of This Structure

1. **Clear wiring**: `index.ts` is the only place that calls `query`/`mutation`.
2. **Feature isolation**: All logic for a feature lives together.
3. **Type safety**: Args are validated with `convex/values`; handlers typed with `QueryCtx`/`MutationCtx`.
4. **Maintainability**: Adding a function is a single new file plus an `index.ts` import/export.

## Implementation Checklist

### When Creating New Functions

- [ ] Add a file in `queries/` or `mutations/` named after the function
- [ ] Export `{FunctionName}Handler` and, if needed, `{FunctionName}Args`
- [ ] Import into `convex/{feature}/index.ts` and declare with `query`/`mutation`
- [ ] Add optional `{FunctionName}Returns` if response validation is desired
- [ ] Verify types and runtime behavior

### When Modifying Existing Functions

- [ ] Update the function file (args/handler/returns as needed)
- [ ] Ensure callers in `index.ts` pass `args` only when defined
- [ ] Adjust any dependent application code as needed

## Example Feature Structures (current codebase)

```
convex/
├── casinos/
│   ├── casinos.model.ts
│   ├── index.ts
│   ├── mutations/
│   │   ├── createCasino.ts
│   │   ├── createMultipleCasinos.ts
│   │   ├── deleteCasino.ts
│   │   └── updateCasino.ts
│   └── queries/
│       ├── getAllCasino.ts
│       ├── getCasinoById.ts
│       └── getCasinosByState.ts
├── states/
│   ├── index.ts
│   ├── states.model.ts
│   ├── mutations/
│   │   ├── createState.ts
│   │   └── getOrCreateState.ts
│   └── queries/
└── offers/
    ├── index.ts
    ├── offers.model.ts
    ├── mutations/
    └── queries/
```

This structure reflects the current project and ensures clean, maintainable, and scalable Convex organization.

---
> Source: [CS-Martin/triumph-2024](https://github.com/CS-Martin/triumph-2024) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
