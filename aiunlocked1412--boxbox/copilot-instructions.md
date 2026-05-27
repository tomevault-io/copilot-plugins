## boxbox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **Claude Code plugin** (not an app). There is no build step, no test framework, no runtime. The repo is just markdown + JSON files that Claude Code loads at install time:

- `.claude-plugin/plugin.json` — manifest (`/plugin install` reads this)
- `.claude-plugin/marketplace.json` — marketplace listing (this repo IS its own marketplace)
- `commands/boxbox.md` — the `/boxbox` slash command body
- `agents/boxbox-{scanner,analyzer,visualizer}.md` — three subagents

The plugin's job: when a user types `/boxbox`, generate `.boxbox/diagram.html` — a beautiful, beginner-friendly system diagram for **non-coders** (vibe-coders, founders, designers).

## Architecture: 3-stage pipeline

`/boxbox` orchestrates three subagents in sequence. Each writes a JSON file consumed by the next:

```
scanner   →  .boxbox/scan.json    (inventory: stack + features + files)
analyzer  →  .boxbox/graph.json   (graph: nodes + edges, all in plain English)
visualizer → .boxbox/diagram.html (self-contained HTML, embeds graph.json)
```

The schemas are **defined inline** in each agent's markdown. If you change a schema field in one agent, you MUST update the consumer agent too — there is no shared schema file.

## Non-obvious invariants

**The HTML template lives inside `agents/boxbox-visualizer.md`.** It is the source of truth for diagram styling. When changing diagram look/feel, edit the visualizer markdown — do not just edit a generated `.boxbox/diagram.html` (that file is throwaway output).

**`/*__GRAPH_JSON__*/` is the only substitution point** in the visualizer template. Adding more template variables means updating the visualizer's "Substitution rule" section to match.

**All plugin-facing text is English.** The audience is global non-coders. Even though the repo owner uses Thai conversationally, every string visible to plugin users (UI labels, agent prompts, headlines, plain_english fields, README) MUST be English.

**"Plain English" is the core UX, not a polish item.** Every output field intended for users (`plain_english`, edge `label`, `summary.headline`, layer names in the legend) must avoid jargon. "User interface" not "Frontend". "Asks the AI to reply" not "POST /api/chat". The agents enforce this — keep it that way when editing.

**Audience reminder lives in the command body** (`commands/boxbox.md`). If you change the audience or tone, update the command body too — that's what Claude reads at runtime to set context for the three agents.

## Testing changes (dry-run)

There is no test framework. To verify a change:

1. Create or use a sample project (`/tmp/boxbox-demo/` is the existing one — a fake Next.js AI chat SaaS exercising all 5 layers and 3 external services).
2. Manually simulate the pipeline: write `scan.json` → `graph.json` → run the visualizer template substitution → produce `diagram.html`.
3. Open in browser: `open /tmp/boxbox-demo/.boxbox/diagram.html`.
4. For a fuller test, install the plugin locally and run `/boxbox` for real:
   ```
   /plugin marketplace add /Users/p1m5/dev-2026/boxbox
   /plugin install boxbox@boxbox-marketplace
   ```

After editing the visualizer template, regenerate the dry-run HTML to verify rendering before commit.

## Diagram tech stack (visualizer)

The generated HTML is self-contained except for three CDN scripts (jsDelivr only):
- `cytoscape@3.30.2` — graph rendering
- `dagre@0.8.5` — hierarchical layout algorithm
- `cytoscape-dagre@2.5.0` — adapter

The layout is **top-down layered** (`rankDir: "TB"`). Layers in vertical order: frontend → api → backend → database. External services float in their own rank. Adding a new layer means updating `LAYER_COLORS` and `LAYER_LABEL` constants in the visualizer template AND the `layer` enum in scanner/analyzer schemas.

## Publishing

The repo at `github.com/aiunlocked1412/boxbox` doubles as its own marketplace. Users install with:

```
/plugin marketplace add aiunlocked1412/boxbox
/plugin install boxbox@boxbox-marketplace
```

Bumping `version` in `plugin.json` is sufficient — the marketplace.json points at `./` (this repo) so a `git push` to main is a release.

---
> Source: [aiunlocked1412/boxbox](https://github.com/aiunlocked1412/boxbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
