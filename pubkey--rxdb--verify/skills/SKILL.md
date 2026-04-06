---
name: verify
description: Verifies code changes by running tests and generation scripts
---

# Verify Changes

This skill verifies that recent code changes are correct and do not break existing functionality.

## Steps

1. Run the fast memory tests to ensure core functionality is working.
   ```bash
   npm run test:fast:memory
   ```



2. (Optional) Run linting to check for style issues.
   ```bash
   npm run lint
   ```

---
> Converted and distributed by [TomeVault](https://tomevault.io) | [Claim this content](https://tomevault.io/claim/pubkey/rxdb)
<!-- tomevault:2.0:skill_md:2026-04-05 -->
