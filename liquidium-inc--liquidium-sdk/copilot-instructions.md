## liquidium-sdk

> When changing public SDK behavior, exported types, method names, request fields, response fields, errors, examples, or runtime configuration:

# Liquidium SDK Agent Instructions

## Documentation

When changing public SDK behavior, exported types, method names, request fields, response fields, errors, examples, or runtime configuration:

- Update the relevant files in `docs/` in the same change.
- Update examples in `examples/` when the documented flow depends on runnable code.
- Keep docs direct and specific. Avoid filler, vague claims, and marketing copy.
- Put user-facing tutorials in `docs/getting-started/`.
- Put task steps in `docs/guides/`.
- Put protocol and SDK explanations in `docs/concepts/`.
- Put lookup material in `docs/api-reference/`.
- Put runnable-app notes in `docs/examples/`.

If exported APIs changed, run `pnpm docs:api` to regenerate `docs/api-reference/generated/`.

---
> Source: [Liquidium-Inc/liquidium-sdk](https://github.com/Liquidium-Inc/liquidium-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
