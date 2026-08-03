## space-io

> - **No `#[allow(dead_code)]`, no `#![allow(dead_code)]`, no `eslint-disable`** unless absolutely unavoidable and explained inline. Don't override, skip, or ignore lints to dodge a warning — fix the code (delete the dead branch, restructure so the helper is actually used, etc.).

# Repo conventions

## Code quality (non-negotiable)

- **No `#[allow(dead_code)]`, no `#![allow(dead_code)]`, no `eslint-disable`** unless absolutely unavoidable and explained inline. Don't override, skip, or ignore lints to dodge a warning — fix the code (delete the dead branch, restructure so the helper is actually used, etc.).
- Same rule for any other linter-suppression attribute (`#[allow(unused_*)]`, `#[allow(clippy::*)]`, `// eslint-disable-line`, `// @ts-ignore`, `// @ts-expect-error`): only with an explicit justification comment, never as a shortcut.
- If you find yourself reaching for an `allow`, prefer restructuring the code or removing the offending symbol.

## Integration tests

- Live under `tests/`. Each test file is a separate cargo binary, so shared helpers must be used by every binary that includes them or moved out of the shared module. We bundle them under a single `tests/integration.rs` entrypoint to keep the helper surface fully exercised.

---
> Source: [nasko05/space-io](https://github.com/nasko05/space-io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
