## githubcdn

> `GithubCDN` is the versioned content payload consumed by the `BibleSpoken` app.

# Repository Guidelines

## Project Purpose
`GithubCDN` is the versioned content payload consumed by the `BibleSpoken` app.

## Critical Docs
- `TEXT_MARKDOWN_PIPELINE.md`
  Open when generating markdown, updating recipe metadata, or preparing data releases.
- `../BibleSpoken/docs/architecture/backward-compatibility-data-and-app.md`
  Open when changing recipe filenames, recipe schema, source fallback behavior, or release compatibility assumptions.

## Compatibility Rule
Do not remove legacy recipe endpoints (`recipe-not-cached.txt`) without an overlap window with newer endpoints (`recipe-not-cached-v2.txt`).

---
> Source: [patrickgeorge1/GithubCDN](https://github.com/patrickgeorge1/GithubCDN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
