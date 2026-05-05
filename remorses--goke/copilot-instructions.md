## goke

> this project uses pnpm.

this project uses pnpm.

use workspace version format for local dependencies.

when publishing start from dependencies first: goke, then mcp, then notion-mcp-cli.

root README is a symlink to goke/README.md

when using schema-based options with `.default()`, never add `(default: X)` in the description string. goke already extracts the default from the schema and appends it to help output automatically. adding it in the description would show the default twice.

---
> Source: [remorses/goke](https://github.com/remorses/goke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
