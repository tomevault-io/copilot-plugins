## pi-whoami

> This repository is `@czottmann/pi-whoami`, a Pi extension package that adds a `whoami` tool to the [pi coding agent](https://github.com/earendil-works/pi-coding-agent). The tool reports the active provider, model, model name, pi thinking level, provider thinking level, and project directory.

# AGENTS.md

## Project overview

This repository is `@czottmann/pi-whoami`, a Pi extension package that adds a `whoami` tool to the [pi coding agent](https://github.com/earendil-works/pi-coding-agent). The tool reports the active provider, model, model name, pi thinking level, provider thinking level, and project directory.

## Important files

- `extensions/whoami.ts` — the extension. Captures thinking fields from provider request payloads and registers the `whoami` tool.
- `README.md` — user-facing install and usage docs, including the field reference.
- `CHANGELOG.md` — per-release notes, shipped in the npm package.
- `package.json` — npm package metadata, Pi manifest, scripts, peer/dev dependencies.
- `.github/workflows/publish.yml` — publishes to npm on a GitHub Release via Trusted Publishing.

## How the extension works

On load, the extension subscribes to `before_provider_request`. For each request it extracts the thinking-related field from the payload (`reasoning_effort`, `reasoning.effort`, `output_config.effort`, `thinking.budget_tokens`, or `generationConfig.thinkingConfig.*`) and caches the last value. The `whoami` tool reads `ctx.model`, `ctx.thinkingLevel`, and `ctx.cwd` at call time and reports the cached wire value as `providerThinkingLevel`. Before the first request of a session, it falls back to the model's `thinkingLevelMap`.

`providerThinkingLevel` is a string for level-based providers and a number for budget-based providers (older Anthropic models, some Google models). It is `null` when the provider received no thinking directive.

## Development commands

```bash
npm run check          # tsc --noEmit
npm pack --dry-run     # for package/release-sensitive changes
```

`npm run build` is an alias for `tsc --noEmit`. This package ships TypeScript source loaded by pi's jiti runtime; there is no compiled `dist/`.

## Coding conventions

- TypeScript is strict, ESM, NodeNext (`tsconfig.json`).
- Keep code simple and explicit. Avoid abstractions without multiple call sites.
- Pi core imports (`@earendil-works/*`, `typebox`) belong in `peerDependencies`/`devDependencies`; do not add runtime dependencies.

## Packaging and releases

- The package ships the source files listed in `files` (`extensions`, `README.md`, `CHANGELOG.md`), not a build. `npm` also includes `package.json` and `LICENSE.md` automatically.
- Releases run through GitHub Releases: add a `CHANGELOG.md` entry, bump the version, commit, tag `vX.Y.Z`, and create a matching GitHub Release. `publish.yml` triggers on `release: published` and runs `npm publish --provenance` via Trusted Publishing.
- Publish a given version either manually or via a GitHub Release, never both — a duplicate publish fails.

## Git hygiene

- Check `git status --short` before committing or broad edits.
- Do not overwrite unrelated user changes.
- Commit only when explicitly asked.

---
> Source: [czottmann/pi-whoami](https://github.com/czottmann/pi-whoami) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
