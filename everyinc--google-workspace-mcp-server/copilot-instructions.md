## google-workspace-mcp-server

> After making changes, bump the version in `package.json` and publish:

# CLAUDE.md

## Publishing to npm

After making changes, bump the version in `package.json` and publish:

1. `npm run build` — compile TypeScript
2. `npm login` — authenticate (must be done manually in terminal by the user)
3. `npm publish` — publish to npm registry

**Note:** `npm publish` requires an active npm login session. Claude cannot run `npm login` (it's interactive). The user must run `npm login` and `npm publish` themselves from the terminal.

---
> Source: [EveryInc/google-workspace-mcp-server](https://github.com/EveryInc/google-workspace-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
