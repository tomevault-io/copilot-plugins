## walletpulse

> TypeScript coding standards and clean code conventions for WalletPulse


# TypeScript Standards

## Type Definitions

- Use `type` for object shapes, unions, intersections
- Use `interface` only when declaration merging is needed (rare)
- Export shared types from `src/shared/types/`
- Colocate component-specific types in the component file

```ts
// Good: union type
type TransactionType = 'income' | 'expense' | 'transfer';

// Good: object shape
type Transaction = {
  id: string;
  amount: number;
  currency: string;
  type: TransactionType;
};

// Bad: avoid enums, use union types instead
enum TransactionType { Income, Expense }
```

## Error Handling

```ts
// Wrap async operations with try/catch and typed errors
async function fetchRates(base: string): Promise<FxRates> {
  try {
    const response = await fetch(`${FX_API_URL}/${base}`);
    if (!response.ok) throw new Error(`FX API ${response.status}`);
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch FX rates:', error);
    throw error;
  }
}
```

## Clean Code Rules

- Strict mode enabled (`"strict": true` in tsconfig)
- No `any` type; use `unknown` with type narrowing
- No non-null assertions (`!`) except in WatermelonDB model decorators
- Prefer `const` over `let`, never use `var`
- Use optional chaining (`?.`) and nullish coalescing (`??`)
- Functions that can fail return `Result<T>` type or throw with descriptive errors
- All currency amounts are `number` type representing cents (integer math only)
- Single responsibility: one function does one thing
- Functions should be under 30 lines; extract helpers if longer
- Maximum 3 parameters per function; use an options object for more
- Name booleans with `is`, `has`, `can`, `should` prefixes
- Name functions with verb-noun pattern: `createTransaction`, `fetchRates`
- Never use em dash characters in strings, comments, or UI text

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kika1s1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
