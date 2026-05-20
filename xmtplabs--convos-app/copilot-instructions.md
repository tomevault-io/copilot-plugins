## imports

> The following contains core principles for writing code in our codebase. Each principle is followed by a small example showing good and bad practices.

The following contains core principles for writing code in our codebase. Each principle is followed by a small example showing good and bad practices.

- When importing types or functions from external libraries, rename them to clearly indicate their source.
- Use the 'as' keyword to rename imports from third-party libraries.

```typescript
// ❌ Bad: Importing without renaming
import { createWallet, Wallet } from "thirdweb/wallets"

// ✅ Good: Renaming imports to indicate source
import {
  createWallet as createWalletThirdweb,
  Wallet as ThirdwebWallet,
} from "thirdweb/wallets"
```

---
> Source: [xmtplabs/convos-app](https://github.com/xmtplabs/convos-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
