## patchright-mcp

> These instructions are for coding agents updating upstream versions or the Patchright dependency.

# Patchright MCP Agent Instructions

These instructions are for coding agents updating upstream versions or the Patchright dependency.

## Update the upstream playwright-mcp version

1) Check the latest upstream version:
   - `npm view @playwright/mcp version`
2) Sync versions across this repo:
   - `package.json` -> `version`
   - `packages/extension/package.json` -> `version`
   - `packages/extension/manifest.json` -> `version`
3) Refresh lockfiles:
   - `npm install --package-lock-only`
   - `npm install --package-lock-only --workspace=packages/extension`
4) Sanity-check:
   - `rg -n "\"version\"" package.json packages/extension/package.json packages/extension/manifest.json` to confirm versions match.

## Update the Patchright version

1) Check the latest Patchright version:
   - `npm view patchright version`
2) Update dependency:
   - `packages/playwright-mcp/package.json` -> `dependencies.patchright`
3) Refresh lockfile:
   - `npm install --package-lock-only`
4) Sanity-check:
   - `rg -n "patchright" packages/playwright-mcp/package.json package-lock.json` to confirm the new version is captured.

## Notes

- Keep the package name as `patchright-mcp` and the CLI bin as `mcp-server-patchright`.
- `PLAYWRIGHT_MCP_EXTENSION_TOKEN` is retained for compatibility with the upstream server.

---
> Source: [Ikaleio/patchright-mcp](https://github.com/Ikaleio/patchright-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
