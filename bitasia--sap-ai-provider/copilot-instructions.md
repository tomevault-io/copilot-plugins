## build-and-publish

> Build, type-check, test, and publish workflow for the package

# Build and Publish

## Build Outputs

- Bundler: tsup (CJS + ESM + DTS + sourcemaps) — see [tsup.config.ts](mdc:tsup.config.ts)
- Output dir: [dist/](mdc:dist/)
- Package exports configured in [package.json](mdc:package.json)

## Commands

- Clean: `npm run clean`
- Build: `npm run build`
- Type-check: `npm run type-check`
- Lint: `npm run lint`
- Prettier check/fix: `npm run prettier-check` / `npm run prettier-fix`
- Test (all): `npm test`

`prepublishOnly` enforces type-check, tests, and build before publish (see [package.json](mdc:package.json)).

## Node Engine

- Requires Node >= 18 (see `engines` in [package.json](mdc:package.json)).

## Verification

- Verify distribution artifacts exist: `npm run check-build`.

---
> Source: [BITASIA/sap-ai-provider](https://github.com/BITASIA/sap-ai-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
