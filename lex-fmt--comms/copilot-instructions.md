## comms

> @.claude/IMPORTANT-RELEASE.md

@.claude/IMPORTANT-RELEASE.md

## Releasing

comms is the base of the lex release chain. Releases are cut through the
fleet release tooling, not by hand-pushing tags.

To cut a release: `release-core cut <major|minor|patch>` (runs in CI — it
dispatches `.github/workflows/release.yml`, the thin caller of the canonical
`gh-action.yml@v3` pipeline). Downstreams pull comms via the git submodule;
there is no automatic push-cascade — bump and release each consumer in
dependency order (the `release-core admin release lex` pipeline does this).

Design + ops + gotchas: [arthur-debert/release/docs/lex-release-cascade.md](https://github.com/arthur-debert/release/blob/main/docs/lex-release-cascade.md).

---
> Source: [lex-fmt/comms](https://github.com/lex-fmt/comms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
