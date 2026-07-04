## noah-for-tinkerers

> This repo is **Noah for Tinkerers** — the open-source, bring-your-own-key build of Noah (a

# AGENTS.md

This repo is **Noah for Tinkerers** — the open-source, bring-your-own-key build of Noah (a
Tauri desktop app that safely fixes computer problems with Claude). The in-app assistant is
named "Noah"; the product/distribution is "Noah for Tinkerers". There is no backend — the app
talks to Anthropic directly with the user's own API key. (The paid product lives in a separate
private repo; don't reintroduce billing/paywall/account code here.)

## Commit Policy
- **Commit after every meaningful change.** Bug fix, feature, refactor — commit it immediately. Don't batch unrelated changes.
- Use conventional commits: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`.
- Keep commits atomic — one logical change per commit.

## Code Style
- Rust: follow existing patterns. `#[cfg]`-gate platform code. Graceful fallback over panics.
- TypeScript/React: functional components, zustand stores, Tailwind classes.
- No over-engineering. Minimum code for the current task.

## Testing
- Run `cargo test --workspace` after Rust changes.
- Run `npx tsc --noEmit` after frontend changes.
- Don't commit code that fails tests or type-checking.

## Version & Release
- App version lives in 3 files — keep in sync: `apps/desktop/src-tauri/tauri.conf.json`, `apps/desktop/package.json`, `apps/desktop/src-tauri/Cargo.toml`. (`crates/noah-tools/Cargo.toml` versions independently — don't touch it.)
- Tag format: `v{VERSION}`. Releases publish as **prereleases** to the BYOK update channel (`onnoah.app/byok`); see CONTRIBUTING.md → "Cutting a release". Use `node scripts/release.mjs` to build + publish.

## Project Structure
- `apps/desktop/src/` — React frontend (Vite + Tailwind)
- `apps/desktop/src-tauri/src/` — Rust backend (Tauri 2)
- `crates/noah-tools/` — shared tool traits
- Platform tools: `src-tauri/src/platform/{macos,windows}/`

---
> Source: [noahapp/noah-for-tinkerers](https://github.com/noahapp/noah-for-tinkerers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
