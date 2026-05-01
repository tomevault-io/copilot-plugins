## no-direct-lockfile-edits

> Never edit the pnpm lockfile manually; use pnpm install


# Lockfile Updates

**CRITICAL**: Never edit `pnpm-lock.yaml` by hand.

When dependency declarations change, run `pnpm install` to update `pnpm-lock.yaml` and `node_modules` together.

If the lockfile changed without running `pnpm install`, run it before continuing.

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
