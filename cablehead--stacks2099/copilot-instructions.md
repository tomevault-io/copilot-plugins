## stacks2099

> **NEVER commit unless explicitly asked by the user.**

## Git Commit Style Preferences

**NEVER commit unless explicitly asked by the user.**

When committing: review `git diff`

- Use conventional commit format: `type: subject line`
- Keep subject line concise and descriptive
- **NEVER include marketing language, promotional text, or AI attribution**
- **NEVER add "Generated with Claude Code", "Co-Authored-By: Claude", or similar
  spam**
- Follow existing project patterns from git log
- Prefer just a subject and no body, unless the change is particularly complex

## Tone and Communication

- ASCII only. No em dashes, smart quotes, or other unicode punctuation. Use
  "--" only in code contexts, not as prose punctuation.
- No wasted words. No fluff. Each word should add value to the reader.
- Calm, matter-of-fact technical tone.

## Code Quality

Always run `./scripts/check.sh` before committing -- it runs the nu unit tests
(`tests/test_*.nu`), `deno fmt --check` (whole repo; scope + excludes in
`deno.json`), `cargo fmt`, clippy, and `cargo test`. Browser e2e lives in
`tests-browser/` (`npm test`). Use `cargo fmt` and `deno fmt` to fix formatting.
nu/sh have no formatter here, so match the existing style by hand.

To see the live UI, screenshot a running instance with `node
tests-browser/shoot.mjs` (defaults to the `--dev` server at 127.0.0.1:5099). See
`tests-browser/README.md`.

## Release Process

Use `/release [version]` to execute the release workflow (version bump, tag,
GitHub Actions build, Homebrew tap).

## Nushell version

When bumping the embedded Nushell (the `nu-*` crate versions in `Cargo.toml`),
update `hustcer/setup-nu`'s `version` in `.github/workflows/ci.yml` to match, so
the `tests/*.nu` suites run on the same engine the binary bundles.

---
> Source: [cablehead/stacks2099](https://github.com/cablehead/stacks2099) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
