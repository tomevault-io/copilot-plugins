## hello-norway

> Always run lint, tests, and build locally before pushing or creating a PR


# Pre-Push Checks

Before pushing code or creating a pull request, **always** run these checks locally in order:

1. `npm run lint` — must exit with 0 errors (warnings are acceptable)
2. `npm run test:unit` — all tests must pass with no regressions introduced
3. `npm run build` — build must succeed

Only proceed to push / create a PR once all three pass cleanly.

---
> Source: [simongottipalli/hello-norway](https://github.com/simongottipalli/hello-norway) — distributed by [TomeVault](https://tomevault.io/claim/simongottipalli).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
