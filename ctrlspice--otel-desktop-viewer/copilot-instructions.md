## otel-desktop-viewer

> - `npm run lint` rejects both Oxlint errors and warnings. Keep the standard lint

# Frontend checks

- `npm run lint` rejects both Oxlint errors and warnings. Keep the standard lint
  gate at zero warnings without disabling rules or adding blanket suppressions.
- `npm run lint:anti-slop:evaluate` reports the production anti-slop cleanup
  backlog; tests and test harnesses are excluded from this advisory scan only.
  Enforced lint, type checks, Vitest, and Playwright retain their existing scope.
- Reviewed external-value decoding and runtime-capability checks may use a
  next-line `anti-slop/no-runtime-typeof` exemption with a concrete boundary
  justification. Scope it to one finding; keep all other findings visible.
- After frontend changes, run `make build-ts` from the repository root and commit
  the updated `desktopexporter/internal/server/static` assets. `make test` runs
  the full local quality gate, including bundle freshness.

---
> Source: [CtrlSpice/otel-desktop-viewer](https://github.com/CtrlSpice/otel-desktop-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
