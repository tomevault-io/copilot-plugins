## anchor-typescript

> Anchor TypeScript

Create unit tests in TS in the `tests` directory. 

Please the node js inbuilt test and assertion libraries (then start the tests using `tsx` instead of `ts-mocha`). Imports look like:

```typescript
import { before, describe, test, it } from "node:test";
import assert from "node:assert";
```

Use `test` rather than `it`.

Use `catch (thrownObject)` and then `const error = thrownObject as Error;` - you can assume any item thrown is an Error.

Use full words. Never use `e` for something thrown or `tx` for a transaction signature. 

---
> Source: [solanakite/anchor-escrow-2026](https://github.com/solanakite/anchor-escrow-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
