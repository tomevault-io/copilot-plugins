## uzumaki

> This repository is a mixed Rust and TypeScript workspace for the Uzumaki UI runtime.

# Repository Guidelines

## Project Structure & Module Organization

This repository is a mixed Rust and TypeScript workspace for the Uzumaki UI runtime.

- `crates/uzumaki/`: main desktop runtime, renderer, Deno integration, and JS bridge code in `src/` and `js/`
- `crates/refineable/`: shared Rust support crate plus `derive_refineable/` proc-macro code
- `packages/playground/`: React-based demo app, entrypoint at `src/index.tsx`
- `scripts/`: small repo utilities such as `copy-binary.ts`

Build artifacts land in `target/`. Top-level config lives in `Cargo.toml`, `package.json`, `pnpm-workspace.yaml`, and `tsconfig.json`.

## Build, Test, and Development Commands

- `pnpm start`: runs the playground through the native runtime (`cargo run -p uzumaki --release src/index.tsx`)
- `cargo build --release -p uzumaki`: builds the desktop runtime
- `cargo test -p uzumaki`: runs the Rust unit tests embedded under `crates/uzumaki/src/`
- `pnpm --filter uzumaki-ui build`: builds the N-API package into `crates/uzumaki/js/bindings`
- `pnpm format`: formats Markdown, TS, JS, and JSON with Prettier

Use `pnpm` for workspace scripts, `cargo` for Rust work, and `bun` only where a package script explicitly requires it.

## Coding Style & Naming Conventions

Rust follows standard `rustfmt` conventions: 4-space indentation, `snake_case` modules/functions, and `PascalCase` types. TypeScript and TSX use Prettier with semicolons and single quotes; current files use 2-space indentation. Name React components in `PascalCase` (`MetricCard`), hooks in `camelCase` (`useInput`), and keep file names lowercase unless a framework convention requires otherwise.

## Rust Validation & Debugging Workflow

After writing or modifying code, always validate the Rust side by running:

- `cargo check`

This ensures everything compiles correctly and catches Rust-side errors early.

In most cases, you do **not** need to run the playground app, as it will typically be handled by the developer. Focus on keeping the Rust code compiling cleanly.

Only run or suggest running the playground (e.g. via `pnpm start`) if the user explicitly asks for debugging, runtime validation, or example behavior verification.

## Testing Guidelines

Rust unit tests live inline in `mod tests` blocks, for example in `crates/uzumaki/src/text.rs` and `crates/uzumaki/src/input.rs`. Add focused tests next to the module you change and run `cargo test -p uzumaki` before opening a PR. There is no root JS test runner configured yet, so treat playground changes as manual-test territory unless you add coverage with the change.

## Commit & Pull Request Guidelines

Recent history uses short, imperative subjects such as `refactor`, `better selection`, and `range provider`. Keep commits brief, specific, and scoped to one change. PRs should explain the behavior change, note affected crates/packages, link any issue, and include screenshots or recordings for playground/UI changes.

---
> Source: [golok727/uzumaki](https://github.com/golok727/uzumaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
