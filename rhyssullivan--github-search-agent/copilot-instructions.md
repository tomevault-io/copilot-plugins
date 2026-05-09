## no-any-usage

> Ban usage of TypeScript's `any` type


Never use TypeScript's `any` type. It defeats the purpose of type safety.

## Alternatives to `any`

- Use `unknown` when the type is truly unknown and needs runtime checks
- Use proper type definitions or generics
- Use `Record<string, unknown>` for objects with unknown structure
- Use union types when multiple types are possible
- Use type assertions (`as Type`) only when absolutely necessary and after proper type narrowing

## Examples

❌ Bad:

```typescript
function processData(data: any) {
  return data.value;
}
```

✅ Good:

```typescript
function processData(data: unknown) {
  if (typeof data === "object" && data !== null && "value" in data) {
    return (data as { value: unknown }).value;
  }
  throw new Error("Invalid data");
}
```

❌ Bad:

```typescript
const result: any = fetchData();
```

✅ Good:

```typescript
const result: ApiResponse = fetchData();
```

If you encounter a situation where you think you need `any`, refactor to use proper types instead.

---
> Source: [RhysSullivan/github-search-agent](https://github.com/RhysSullivan/github-search-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
