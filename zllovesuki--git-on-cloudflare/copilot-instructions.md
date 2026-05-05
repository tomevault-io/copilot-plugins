## wrangler

> 1. To add new variables/binding, first make changes in wrangler.jsonc, then run `npm run cf-typegen` to update `worker-configuration.d.ts`. The new bindings will be available via `Env`


1. To add new variables/binding, first make changes in wrangler.jsonc, then run `npm run cf-typegen` to update `worker-configuration.d.ts`. The new bindings will be available via `Env`
2. Do not attempt to read `worker-configuration.d.ts` in full as it is auto-generated, and the line limit exceeds what your tool allows.

---
> Source: [zllovesuki/git-on-cloudflare](https://github.com/zllovesuki/git-on-cloudflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
