## ocean-node

> Run nvm use before tests to avoid SyntaxError (e.g. from "with" or other Node-version-dependent syntax)


# Running tests

Before running any test commands in this project (e.g. `npm test`, `npm run mocha`, `npm run test:unit`), run:

```bash
nvm use
```

This ensures the Node version from `.nvmrc` is active. Skipping it can cause SyntaxError or other version-related failures.

---
> Source: [oceanprotocol/ocean-node](https://github.com/oceanprotocol/ocean-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
