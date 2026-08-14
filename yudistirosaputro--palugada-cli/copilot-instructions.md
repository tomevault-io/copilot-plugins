## palugada-cli

> <!-- palugada:start -->

<!-- palugada:start -->
# Working with palugada

This project uses **palugada** for token-cheap, always-current engineering
context — ask palugada instead of re-reading lots of files.

**Before** grepping for code (`grep`/`find`/`rg`/Glob), use the index:
`palugada symbol <name>` / `palugada fact <family>`.

On-demand skills (loaded by trigger):
- `palugada-search` — locate code/symbols (use before grep)
- `palugada-bugfix` / `-feature` / `-refactor` / `-review` — scoped task packs via `palugada brief`
- `palugada-figma-to-code` — implement UI from Figma via the figma-bridge MCP (free-plan safe)
- `palugada-postman-api` — read Postman collections via the postman-bridge MCP (offline-first)

Discover: `palugada q --list` (conventions) · `palugada for --list` (recipes) · `palugada <cmd> --help`.
Bound profile: `rust-cli` — switch with `palugada profile use <id>` (skills follow the active profile automatically).
<!-- palugada:end -->

# MCP bridges (this repo)

`.mcp.json` registers two local quota-guarding MCP servers from `mcp/`:
- **figma-bridge** — Figma node trees/styles for UI-to-code, cache-first
  (free plan ≈ 6 file fetches/month). Skill: `palugada-figma-to-code`.
- **postman-bridge** — read Postman collections/environments offline-first
  (free plan ≈ 300 API calls/month). Skill: `palugada-postman-api`.

After editing `mcp/*/src`, rebuild (`npm run build`) and run `node mcp/smoke.mjs`.
`palugada init` scaffolds both skills into every project (src/scaffold.rs).
The web console's Bridges tab (`palugada web`) shows quota/cache state.

# Auto-index git hooks (this repo)

`palugada hook install` (src/hook.rs) writes `.git/hooks/{post-commit,post-merge,post-checkout,post-rewrite}`
that re-`index` in the background when HEAD moves — the exact events that make the
index stale (`indexer::staleness_note` compares HEAD to the indexed sha). Blocks
are marked (`# >>> palugada auto-index >>>`) so they coexist with other hooks and
uninstall cleanly. `palugada init --hooks` installs them during scaffold (opt-in).

---
> Source: [yudistirosaputro/palugada-cli](https://github.com/yudistirosaputro/palugada-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
