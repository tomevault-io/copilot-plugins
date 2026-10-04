## ryostore

> This repo has a Prowl index of its files, symbols, and how they connect. To find

<!-- prowl-agent -->
## Prowl project context

This repo has a Prowl index of its files, symbols, and how they connect. To find
code, read one symbol, trace who calls it, or check what a change touches,
**query Prowl first** -- do not grep or read whole files just to locate things.
Prowl reindexes what changed before each query, so answers stay current, and it
returns ranked, cited file:line results in one call instead of a grep hit list
you then open files to disambiguate.

The same index is reachable two ways, and every agent should know both:

- **Preferred -- Prowl MCP tools** (they appear in your tool list when your
  harness wires Prowl as an MCP server): search_context (where/how does X work),
  read_symbol (one symbol's source), outline (a file's structure, no bodies),
  find_references (call sites), analyze_change (blast radius before an edit), and
  sketch_ui (how a UI looks, from source).
- **Opt-in -- the prowl-agent CLI** (same capabilities, for when MCP is not
  wired): overview, find <name>, outline <path>, def <name>, references <name>,
  callers|callees|relations <path>, impact <path>, search <text>, wip, and
  changed / doctor after edits.

Keep grep and glob for literal-string and filename scans only. CLI output is
token-lean TOON by default; add --format human|toon|json|markdown.
<!-- /prowl-agent -->

<!-- prowl-agent:map -->
## Prowl project map

Auto-generated from the Prowl index, refreshed on each `overview`/`init`. Prefer retrieving from Prowl (and reading the cited files) over grepping or relying on training memory; this is the current shape of the repo.

- size: 401 files, 19947 symbols, 1100 edges (resolved 454, external deps 65, unresolved 581)
- languages: json:164 qml:153 generic:36 bash:20 markdown:17 python:7 javascript:3 yaml:1
- subsystems: barstyles/nacre(31,qml) · barstyles/obi(28,qml) · plugins/market(17,qml) · plugins/obsidian(15,qml) · plugins/stargate(12,qml) · lockscreens/genshin(5,qml) · plugins/photo-frame(5,qml) · bundles/the-influencer(4,bash)
- entrypoints: rices/registry.json · barstyles/nacre/Scene.qml · barstyles/nacre/manifest.json · barstyles/obi/Scene.qml · barstyles/obi/manifest.json · bundles/the-influencer/bundle.json · lockscreens/clockwork-tape/manifest.json · lockscreens/dog-samurai/manifest.json · (+44 more)
- central files (most depended-on): plugins/obsidian/content/Eyebrow.qml · plugins/stargate/content/GateGlyph.qml · plugins/photo-frame/content/PhotoFrame.qml · plugins/market/content/Sparkline.qml · lockscreens/terraria/content/TerraButton.qml
- read these guides first: README.md · CONTRIBUTING.md

Depth on demand: `prowl-agent find|def|outline|references <name>`, `search <text>`, `context search "<question>"`, `sketch <ui>`.
<!-- /prowl-agent:map -->

---
> Source: [Ryoku-dev/ryostore](https://github.com/Ryoku-dev/ryostore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
