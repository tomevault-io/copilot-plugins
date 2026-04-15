## stream-llm

> - Use kebab-case for file names (e.g., `my-component.ts`)

- Use kebab-case for file names (e.g., `my-component.ts`)
- Use camelCase for variables and function names (e.g., `myVariable`, `myFunction()`)
- Use UpperCamelCase (PascalCase) for classes, types, and interfaces (e.g., `MyClass`, `MyInterface`)
- Use ALL_CAPS for constants and enum values (e.g., `MAX_COUNT`, `Color.RED`)
- Inside generic types, functions or classes, prefix type parameters with `T` (e.g., `TKey`, `TValue`)

```ts
type RecordOfArrays<TItem> = Record<string, TItem[]>;
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amit-t) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
