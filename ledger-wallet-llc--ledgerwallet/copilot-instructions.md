## prefer-set-has

> Prefer Set.has() over Array.includes() for constant allowlists/blocklists


# Prefer Set .has() over Array .includes()

When checking membership in a **constant list** of known values (allowlists, blocklists, enum-like sets), use a `Set` with `.has()` instead of an array with `.includes()`.

```typescript
// ❌ BAD
const ALLOWED = ["a", "b", "c"];
if (ALLOWED.includes(value)) { … }

// ✅ GOOD
const ALLOWED = new Set(["a", "b", "c"]);
if (ALLOWED.has(value)) { … }
```

**Why:** `Set.has()` is O(1) vs `.includes()` O(n), communicates "membership test" intent more clearly, and avoids accidental mutation of the backing array.

**When .includes() is fine:** Searching within a dynamic or short-lived array (e.g. function parameters, user input) where creating a Set would add noise.

---
> Source: [Ledger-Wallet-LLC/ledgerwallet](https://github.com/Ledger-Wallet-LLC/ledgerwallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
