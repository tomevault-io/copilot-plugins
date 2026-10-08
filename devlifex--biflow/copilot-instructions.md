## agents

> Follow repository AGENTS.md for tests, ADRs, lessons, and versioning


# Project agent rules

Read and follow `AGENTS.md` in the repository root.

- Keep e2e coverage for primary flows and unit tests for core and UI.
- After every change, prove only what changed with zero failures and zero project warnings: frontend `pnpm check` + `pnpm build` when UI/scripts change; Rust `cargo test -p <crate>` plus `cargo clippy -p <crate> --all-targets -- -D warnings` for each touched crate, then `cargo fmt --all --check` before calling the change done. Do not `cargo clean` or rebuild the whole workspace every task.
- User-visible UI changes recapture `docs/screenshots/` with `BIFLOW_CAPTURE_README=1 pnpm exec playwright test e2e/readme-screenshots.spec.ts` in the same change.
- After fixing an error, append a lesson to `AGENTS.md`.
- Keep `./dev.sh` native and Rust-backed by default; browser/mock mode must be explicit as `./dev.sh web`.
- Add or update an ADR in `docs/adr/` for each behavioral change.
- Bump only the root `version` file, then run `pnpm version:sync`.

---
> Source: [devlifeX/BiFlow](https://github.com/devlifeX/BiFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
