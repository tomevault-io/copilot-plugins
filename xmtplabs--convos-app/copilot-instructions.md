## function-parameters

> The following contains core principles for writing code in our codebase. Each principle is followed by a small example showing good and bad practices.

The following contains core principles for writing code in our codebase. Each principle is followed by a small example showing good and bad practices.

- Use object parameters with inline types.
- Destructure parameters at the top of the function body.

## Examples

```typescript
// ❌ Bad: Individual parameters
function sendMessage(recipientId: string, content: string) {
  // Function body
}

// ✅ Good: Object parameters with destructuring
function sendMessage(args: { recipientId: string; content: string }) {
  const { recipientId, content } = args
  // Function body
}
```

---
> Source: [xmtplabs/convos-app](https://github.com/xmtplabs/convos-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
