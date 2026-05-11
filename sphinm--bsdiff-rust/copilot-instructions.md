## bsdiff-rust

> This is a Rust + Node.js native addon library (`@bsdiff-rust/node`) using NAPI-RS. It implements bsdiff/bspatch binary diff/patch algorithms. There are no external services (no databases, no HTTP servers, no Docker required for local dev).

## Cursor Cloud specific instructions

This is a Rust + Node.js native addon library (`@bsdiff-rust/node`) using NAPI-RS. It implements bsdiff/bspatch binary diff/patch algorithms. There are no external services (no databases, no HTTP servers, no Docker required for local dev).

### Key commands

All standard dev commands are in `package.json` scripts — see README.md for details. Quick reference:

| Task | Command |
|------|---------|
| Install deps | `pnpm install` |
| Build native addon (release) | `pnpm build` |
| Build native addon (debug) | `pnpm build:debug` |
| Run tests | `pnpm test` |
| Lint | `pnpm lint` |
| Format | `pnpm format` |
| Benchmark | `pnpm run bench` |

### Non-obvious caveats

- **Rust version**: The napi v3 dependencies require Rust >= 1.88. The pre-installed Rust toolchain may be too old. Run `rustup default stable` after `rustup update stable` if `pnpm build` fails with `edition2024` or version errors.
- **Test resources**: Tests require two zip files that are NOT checked into git. They must be downloaded before running `pnpm test`:
  ```
  curl -L -o test/resources/react-18.1.0.zip "https://github.com/facebook/react/archive/refs/tags/v18.1.0.zip"
  curl -L -o test/resources/react-19.1.0.zip "https://github.com/facebook/react/archive/refs/tags/v19.1.0.zip"
  ```
- **`cargo fmt --check`** will report diffs because the Rust source uses 4-space indentation while `rustfmt.toml` specifies `tab_spaces = 2`. This is a pre-existing inconsistency in the repo.
- The native `.node` binary is generated in the workspace root (e.g., `node.linux-x64-gnu.node`) and is loaded by `index.js`. You must run `pnpm build` before tests or any usage.
- `lint-staged` + `husky` are configured in `package.json` but the `.husky/` directory is absent, so git hooks are not active.

---
> Source: [Sphinm/bsdiff-rust](https://github.com/Sphinm/bsdiff-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
