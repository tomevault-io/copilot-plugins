## sage

> Protect generated bindings and release-time translation catalogs


# Generated Artifacts

- Never edit `src/bindings.ts` manually. Change its Rust/Specta source definitions instead.
- Never run Sage automatically to regenerate `src/bindings.ts`; Sage is a GUI application.
- Regenerate bindings with `pnpm generate:bindings` when needed; this command must not launch Sage.
- Release builds must never generate or modify `src/bindings.ts`.
- Do not run `pnpm extract` or modify `src/locales/**/*.po` during normal feature work.
- Update `.po` catalogs only when explicitly requested, typically as part of release preparation.
- If a task requires generated artifacts to be refreshed, report that follow-up rather than hand-editing them.

---
> Source: [xch-dev/sage](https://github.com/xch-dev/sage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
