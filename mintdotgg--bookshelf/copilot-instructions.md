## bookshelf

> - For Three.js, generated-asset, model-viewer, material, animation, or world

# Repository guidance

## Mint tooling

- For Three.js, generated-asset, model-viewer, material, animation, or world
  work, start with the repo-local `mint-threejs-skills` skill at
  `.agents/skills/mint-threejs-skills/SKILL.md` and follow its routing.
- Use the repo-scoped `mint` MCP server as the generated-asset production
  pipeline. MCP generation calls are development actions and must never be
  added to browser runtime code.
- Never commit Mint credentials, OAuth state, private account data, or temporary
  download URLs.
- When Mint assets are imported, keep the project-root `mint-assets.json`
  registry in sync with the vendored `scripts/sync-mint-assets.mjs` helper.
- Mint-optimized GLBs require the shared Draco-capable loader described by the
  vendored skill. Do not load them with a bare `GLTFLoader`.
- Preserve the existing application architecture and keep provider branding,
  asset IDs, and provenance out of the product UI unless a product requirement
  explicitly calls for them.

Run `npm run check:mint` after changing the MCP configuration or vendored skill
suite.

---
> Source: [mintdotgg/bookshelf](https://github.com/mintdotgg/bookshelf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
