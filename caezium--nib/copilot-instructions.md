## nib

> This app's desktop framework ships its own API docs, and they're more current than any

# Read the framework docs before writing code

This app's desktop framework ships its own API docs, and they're more current than any
model's training data — its APIs get renamed and reorganized, so code written from memory
won't compile.

**Before writing or modifying framework code**, read the documentation under
`node_modules/@mobrowser/api/docs/`:
- `node_modules/@mobrowser/api/docs/guides/` — architecture, project structure, the
  multi-process model, IPC, the native module, features, and examples.
- `node_modules/@mobrowser/api/docs/api/` — the API reference with code examples.

Don't guess API names, method signatures, or import paths — look them up in the docs.
If the `docs/` directory is missing, run `npm run gen` to download it.

---
> Source: [caezium/nib](https://github.com/caezium/nib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
