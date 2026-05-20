## consola

> This project uses consola for logging with a globally configured log level.


# Logging

- Use `consola` (from `consola`), not `console`. Log level is set from runtime config in `app/plugins/01.logging.ts`. In tests that touch code using consola, mock `consola` (see `test/nuxt/app/plugins/01.logging.test.ts`).

---
> Source: [Wetzel402/Skylite-UX](https://github.com/Wetzel402/Skylite-UX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
