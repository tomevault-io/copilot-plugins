## koowerk

> Use JSDoc comments to annotate functions and types.

Use JSDoc comments to annotate functions and types.

Be concise in JSDoc comments, and only provide JSDoc comments if the function's behaviour is not self-evident.

Use the JSDoc inline `@link` tag to link to other functions and types within the same file.

```ts
/**
 * Subtracts two numbers
 */
const subtract = (a: number, b: number) => a - b;

/**
 * Does the opposite to {@link subtract}
 */
const add = (a: number, b: number) => a + b;
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ehoneahobed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
