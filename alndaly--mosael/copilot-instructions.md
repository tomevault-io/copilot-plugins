## mosael

> When `.codegraph/` exists, use `codegraph explore` (or its MCP equivalent) before text searches to locate or understand code. Recheck the current branch and local changes at the start of work; preserve updates made by other tools.

# Mosael repository instructions

## Understand current code

When `.codegraph/` exists, use `codegraph explore` (or its MCP equivalent) before text searches to locate or understand code. Recheck the current branch and local changes at the start of work; preserve updates made by other tools.

## Packaging and releases

When the user asks to build or publish a release, read [docs/RELEASING.md](docs/RELEASING.md) and [docs/MACOS_SIGNING.md](docs/MACOS_SIGNING.md) first. They document the actual GitHub Actions and local macOS notarization handoff, including the existing `mosael-release` Keychain profile. Use existing credentials without printing or exporting secrets. A release is complete only when the tested installers are verified and the authorized GitHub Release is published; pushing code or dispatching a workflow alone is not completion.

---
> Source: [Alndaly/Mosael](https://github.com/Alndaly/Mosael) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
