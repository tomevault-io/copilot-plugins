## mcp-registry-tools

> FastMCP server for searching and resolving packages on the mpak registry.

# Registry Tools MCP Server

FastMCP server for searching and resolving packages on the mpak registry.

## Commands

```bash
make test               # Unit tests
make check              # Format + lint + typecheck + unit tests
make bump VERSION=x.y.z # Bump version in all files
```

## Release

See `mcp-servers/CLAUDE.md` for the full release workflow.

```bash
make bump VERSION=0.2.0
git add -A && git commit -m "Bump version to 0.2.0"
git tag v0.2.0 && git push origin main v0.2.0
gh release create v0.2.0 --title "v0.2.0" --notes "- changelog"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NimbleBrainInc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NimbleBrainInc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
