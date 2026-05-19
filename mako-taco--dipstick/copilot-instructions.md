## dipstick

> When creating unit tests that require ts-morph structures, do not mock them.

# Unit testing
When creating unit tests that require ts-morph structures, do not mock them.
Instead, create a `__fixtures__` directory in the same directory as the test,
and add `.ts` source files to it, which can then be parsed by ts-morph.

---
> Source: [mako-taco/dipstick](https://github.com/mako-taco/dipstick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
