## native-env

> Environment access — only through env.ts


# Environment variables — `env.ts` only

**Do not use `process.env` or `import.meta.env` anywhere except `apps/native/src/lib/env.ts`.**

All app code reads deployment settings through exports from that module (`settings`, `nixmacEnvironment`, `getProfileValue`, etc.).

Benefits: single validation path, typed profile JSON, no scattered env reads, and build-time profile baking stays consistent with Rust (`build.rs`).

```typescript
// ❌ BAD
const key = import.meta.env.VITE_POSTHOG_KEY;

// ✅ GOOD
import { settings } from "@/lib/env";
const key = settings.posthogKey;
```

---
> Source: [darkmatter/nixmac](https://github.com/darkmatter/nixmac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
