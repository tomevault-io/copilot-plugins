## use-timeouts-helper-in-tests

> Use shared timeout helpers in tests


# Use Timeouts Helper In Tests

When a test needs an explicit timeout, always use a shared named helper from `@prisma-next/test-utils` (for example `timeouts.spinUpPpgDev`).

## Do

- Use `timeouts.*` constants instead of raw numbers.
- Keep timeout values centralized in `@prisma-next/test-utils`.

## Don't

- Don't write raw numeric timeout values in test bodies (for example `{ timeout: 1_000 }`).

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
