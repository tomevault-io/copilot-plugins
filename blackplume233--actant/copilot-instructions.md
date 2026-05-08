## ship-testing

> Use incremental testing during /trellis-ship instead of full test suite


# Ship Testing Convention

During `/trellis-ship` (Phase 1 review), run **incremental tests** instead of the full suite:

- Use `pnpm test:changed` (vitest `--changed`) which only runs tests related to uncommitted changes
- Only fall back to `pnpm test` (full suite) when explicitly requested or when changes touch shared infrastructure (e.g. vitest.config.ts, tsconfig, package.json dependencies)

---
> Source: [blackplume233/Actant](https://github.com/blackplume233/Actant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
