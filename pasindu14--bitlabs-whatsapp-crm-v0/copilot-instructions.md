## bitlabs-whatsapp-crm-v0

> Apply this rule to anything in the Actions layer (features/**/actions/*-actions.ts): Next.js server actions wrapped with withAction, validation, service delegation, auth context, and Result<T> unwrapping.


# Actions Rules

<actions_rules>

## Applies to
- `features/**/actions/*-actions.ts` (Next.js server actions)

## Hard requirements
- All actions wrapped with `withAction<>()` helper
- Actions receive `auth` (AuthSession) and `input` (validated) automatically
- All actions return `Result<T>` (never throw for expected failures)
- Validate input using schema in `withAction` options
- Extract `companyId` from `auth` and pass to services

## Action structure
```typescript
export const {actionName}Action = withAction<InputType, ResponseType>(
  "domain.action",  // action name for logging
  async (auth, input) => {
    // 1. Call service
    const result = await Service.method({
      ...input,
      companyId: auth.companyId,
      userId: auth.userId,  // if needed for audit
    });

    // 2. Unwrap Result
    if (!result.success) {
      return Result.fail(result.message, result.error);
    }

    // 3. Return success
    return result;
  },
  { schema: validationSchema }  // client or server schema
);
```

## Schema selection
- Use **client schemas** for public actions (user-facing forms)
- Use **server schemas** for internal actions (require auth context)
- Never validate manually - let `withAction` handle it
- Example:
  ```typescript
  // Public action (form submission)
  { schema: userCreateClientSchema }

  // Internal action (requires auth)
  { schema: userGetServerSchema }  // includes companyId/userId
  ```

## Auth context
- `auth` is automatically injected by `withAction`
- Always use `auth.companyId` for multi-tenancy
- Use `auth.userId` as actor for audit logs
- Never manually extract session - use provided `auth`

## Error handling
- **NEVER use try/catch in actions** - Result<T> handles all errors
- Unwrap service results with `if (!result.success) return Result.fail(...)`  
- Preserve original error messages from services
- Return failures immediately (no fallback logic)

## Service delegation
- Actions are thin wrappers - no business logic
- All logic lives in services
- One action = one service method (typically)
- For complex workflows, create orchestrator services

## Action categories
- **List actions**: `list{Domain}Action` - return paginated lists
- **Get actions**: `get{Domain}Action` - return single entity
- **Create actions**: `create{Domain}Action` - create new entity
- **Update actions**: `update{Domain}Action` - update existing entity
- **Delete actions**: `delete{Domain}Action` - soft/hard delete
- **Custom actions**: `{action}{Domain}Action` - domain-specific operations

## Naming conventions
- Action exports: `{actionName}Action` (e.g., `listUsersAction`, `createUserAction`)
- Action names in `withAction`: `"domain.action"` (e.g., `"users.list"`, `"users.create"`)
- Use kebab-case for action names in strings
- Use PascalCase for action exports

## Boundaries
- No direct DB queries in actions (use services)
- No business logic in actions (delegate to services)
- No external API calls in actions (use services)
- Actions orchestrate validation + service calls only

## Type safety
- Import types from schemas: `z.infer<typeof schema>`
- Import response types from schemas: `ResponseType`
- Never use `any` - always infer from schemas
- Example:
  ```typescript
  import type { UserCreateInput } from "../schemas/user-schema";
  import type { UserResponse } from "../schemas/user-schema";
  ```

## Performance
- Actions are server-side - no client-side concerns
- `withAction` handles logging and error tracking
- Keep actions synchronous where possible
- For async operations, ensure proper error propagation

## Security
- Never expose internal errors to client
- Validate all input via schemas
- Always filter by `companyId` from auth
- Never trust client-provided `companyId`/`userId`
- **Client schemas should NOT include `companyId`/`userId`** - these come from auth session

## Exports
- Use barrel file (`index.ts`) when exporting multiple actions from a directory
- Re-export all actions from the barrel file for clean imports
- Example:
  ```typescript
  // features/users/actions/index.ts
  export { listUsersAction } from './user.actions';
  export { getUserAction } from './user.actions';
  export { createUserAction } from './user.actions';
  export { updateUserAction } from './user.actions';
  export { deleteUserAction } from './user.actions';
  ```

</actions_rules>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pasindu14) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
