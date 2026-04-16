## authentication-security-levels-system

> TypeScript best practices and conventions


# TypeScript Best Practices

## Configuration

- Use `strict` mode in `tsconfig.json`
- Enable `strictNullChecks`, `strictFunctionTypes`
- Use `noImplicitAny`, `noUnusedLocals`, `noUnusedParameters`

## Code Patterns

### Async/Await

- Prefer `async/await` over Promise chains
- Handle errors with try/catch
- Use Promise.all for parallel operations

### Optional Chaining

- Use optional chaining (`?.`) for safe property access
- Use nullish coalescing (`??`) for default values

### Destructuring

- Use destructuring for cleaner code
- Provide default values in destructuring

## DO

- ✅ Use strict TypeScript configuration
- ✅ Use async/await for asynchronous code
- ✅ Use optional chaining and nullish coalescing
- ✅ Use const assertions for immutable data
- ✅ Leverage type inference where appropriate

## DON'T

- ❌ Use `any` type
- ❌ Use Promise chains when async/await is clearer
- ❌ Ignore TypeScript compiler warnings
- ❌ Use type assertions to bypass type checking

## Examples

### Good: Modern TypeScript

```typescript
interface Config {
  apiUrl?: string;
  timeout?: number;
}

function getApiUrl(config?: Config): string {
  return config?.apiUrl ?? 'https://api.example.com';
}
```

### Bad: Older Patterns

```typescript
function getApiUrl(config: any): string {
  if (config && config.apiUrl) {
    return config.apiUrl;
  }
  return 'https://api.example.com';
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlinRosca10)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AlinRosca10)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
