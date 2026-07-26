## anansi

> Package Development Workflow


When modifying files in `packages/*`:

1. **Build after changes**: Run `yarn build` from the package directory (or use workspace command from root):
   ```bash
   yarn workspace <package-name> run build
   ```

2. **Test consumers**: After building, test changes in an example app like `examples/concurrent` by running `yarn start` from that directory.

---
> Source: [ntucker/anansi](https://github.com/ntucker/anansi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
