## bytescale-javascript-sdk

> When the user asks to release the Bytescale JavaScript SDK:

# Repository Instructions

## Releasing

When the user asks to release the Bytescale JavaScript SDK:

1. Read and follow `RELEASING.md`.
2. Unless the user specifies another version, bump the minor version and reset the patch version to zero.
3. Perform the release only by updating the version in `package.json`, committing on `main` with the message `Release x.y.z`, and pushing `main`.
4. Do not create the release tag or publish to npm locally. The GitHub Action creates the tag and publishes the package after recognizing the release commit.

---
> Source: [bytescale/bytescale-javascript-sdk](https://github.com/bytescale/bytescale-javascript-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
