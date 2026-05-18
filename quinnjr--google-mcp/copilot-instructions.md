## typescript

> TypeScript best practices and coding standards


# TypeScript Best Practices

Follow these conventions when writing TypeScript code in this project.

## Type Safety

1. **Avoid `any`** - Use `unknown` for truly unknown types, then narrow with type guards
   ```typescript
   // Bad
   function process(data: any) { ... }

   // Good
   function process(data: unknown) {
     if (typeof data === 'string') { ... }
   }
   ```

2. **Prefer explicit return types** - Always declare return types on functions and methods
   ```typescript
   // Bad
   function getData() { return { id: 1 }; }

   // Good
   function getData(): { id: number } { return { id: 1 }; }
   ```

3. **Use strict null checks** - Handle `null` and `undefined` explicitly
   ```typescript
   // Bad
   const name = user.name.toUpperCase();

   // Good
   const name = user.name?.toUpperCase() ?? 'Unknown';
   ```

## Access Modifiers

All class members must have explicit access modifiers:

```typescript
class MyService {
  private readonly client: Client;

  constructor(client: Client) {
    this.client = client;
  }

  public async getData(): Promise<Data> { ... }

  private validateInput(input: string): boolean { ... }
}
```

- Use `private` for internal implementation details
- Use `public` for API surface
- Use `protected` for inheritance hierarchies
- Use `readonly` for immutable properties

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Classes | PascalCase | `UserService` |
| Interfaces | PascalCase | `UserConfig` |
| Type aliases | PascalCase | `UserId` |
| Enums | PascalCase | `UserRole` |
| Enum members | UPPER_CASE or PascalCase | `ADMIN` or `Admin` |
| Functions | camelCase | `getUserById` |
| Variables | camelCase | `userCount` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Private members | camelCase (no underscore prefix) | `private data` |

## Type Imports

Use type-only imports when importing types:

```typescript
// Bad
import { User, UserService } from './user';

// Good
import { type User, UserService } from './user';
// or
import type { User } from './user';
import { UserService } from './user';
```

## Async/Await

1. **Always handle promises** - Never leave promises floating
   ```typescript
   // Bad
   someAsyncFunction();

   // Good
   await someAsyncFunction();
   // or
   void someAsyncFunction(); // explicitly ignore
   ```

2. **Use async/await over .then()** - More readable and easier to debug
   ```typescript
   // Bad
   getData().then(data => process(data)).catch(err => handle(err));

   // Good
   try {
     const data = await getData();
     await process(data);
   } catch (err) {
     handle(err);
   }
   ```

## Error Handling

1. **Type catch clause variables as `unknown`**
   ```typescript
   try {
     await riskyOperation();
   } catch (error: unknown) {
     if (error instanceof Error) {
       console.error(error.message);
     }
   }
   ```

2. **Create custom error classes for domain errors**
   ```typescript
   class AuthenticationError extends Error {
     constructor(message: string) {
       super(message);
       this.name = 'AuthenticationError';
     }
   }
   ```

## Nullish Coalescing and Optional Chaining

Prefer `??` over `||` for default values (handles `0` and `''` correctly):

```typescript
// Bad - treats 0 and '' as falsy
const count = input || 10;

// Good - only treats null/undefined as nullish
const count = input ?? 10;
```

Use optional chaining for nested access:

```typescript
// Bad
const city = user && user.address && user.address.city;

// Good
const city = user?.address?.city;
```

## Immutability

1. **Use `readonly` for array/object properties that shouldn't change**
   ```typescript
   interface Config {
     readonly endpoints: readonly string[];
   }
   ```

2. **Use `as const` for literal types**
   ```typescript
   const ROLES = ['admin', 'user', 'guest'] as const;
   type Role = typeof ROLES[number]; // 'admin' | 'user' | 'guest'
   ```

## Generics

Use meaningful generic names:

```typescript
// Bad
function transform<T, U>(input: T): U { ... }

// Good
function transform<TInput, TOutput>(input: TInput): TOutput { ... }
```

## Documentation

Use JSDoc for public APIs:

```typescript
/**
 * Fetches user data from the API.
 * @param userId - The unique identifier of the user
 * @returns The user object if found
 * @throws {NotFoundError} When user doesn't exist
 */
public async getUser(userId: string): Promise<User> { ... }
```

---
> Source: [quinnjr/google-mcp](https://github.com/quinnjr/google-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
