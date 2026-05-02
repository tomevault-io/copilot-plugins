## build-and-package

> Build and packaging instructions for the Cursor RTL extension


# Build & Package

- Always use `npm run package` to create a VSIX. It runs build + packaging in one step.
- Never run `vsce package` directly — the `package` script ensures the dist is rebuilt first and uses `--no-dependencies` since all deps are bundled via esbuild.

---
> Source: [motcke/cursor-ext-rtl](https://github.com/motcke/cursor-ext-rtl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
