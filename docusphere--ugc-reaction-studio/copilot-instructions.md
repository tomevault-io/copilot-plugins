## ugc-reaction-studio

> A generate-and-composite pipeline for talking-head reaction ads. One skill takes a product to a finished ad: an AI creator (corner or full-screen) over a main screen that cuts between talking selfies, b-roll, product shots, and real screenshots — generated via Higgsfield MCP, composited in Remotion, all inside Claude Code.

# UGC Reaction Ad Studio — project guide

A generate-and-composite pipeline for talking-head reaction ads. One skill takes a product to a finished ad: an AI creator (corner or full-screen) over a main screen that cuts between talking selfies, b-roll, product shots, and real screenshots — generated via Higgsfield MCP, composited in Remotion, all inside Claude Code.

- **`/ugc-reaction-studio`** — the full pipeline: setup/inputs → script/beats → config → creator → clips → VO → screenshots → Remotion composite/render.

## Workspace: inputs/ and outputs/
The skill creates and works in two folders — never ask the user to make folders/files by hand:
- **`inputs/`** — what the user feeds in: `config.csv`, `beats.csv`, `product.png`, optional `creator-ref.png`, and `screenshots/` (captured evidence, `beat-N-…`).
- **`outputs/`** — what the skill produces: `creator/`, `clips/` (`beat-N-…-.mp4`), `vo/`, `remotion/`, `final/`.

## How to work in this project
- On session start, run **`/ugc-reaction-studio`** and let its opening message be the ONLY welcome.
- **First run scaffolds `inputs/` + `outputs/`** and asks the user for inputs (product details + image, an existing Google Sheet exported to `inputs/config.csv` + `inputs/beats.csv`, optional creator reference). If `inputs/config.csv` + `inputs/beats.csv` already exist, read them instead of starting blank.
- When the user wants a reaction-style ad, an explainer with on-screen proof, or a talking-head-over-b-roll ad → use `/ugc-reaction-studio`.
- **Generation runs through Higgsfield MCP** (creator + clips + product shots) → `outputs/`. **Compositing + render run in Remotion** → `outputs/final/`. Files land in the workspace — never ask the user to paste prompts into another tool. If Higgsfield isn't connected, or Remotion isn't set up, resolve that first.
- **Story-first, gated order:** setup/inputs → script/beats → config → creator → clips → VO → screenshots → composite. Don't skip or combine.
- **Consistency:** lock the creator once, then attach that creator reference (and `inputs/product.png`) to EVERY generated clip so the same person + product appear throughout.
- **Prompts:** the user authors the creative columns; Claude writes the per-beat generation prompt into the `Gen Prompt` column for review before generating.
- **The evidence layer is real** — screenshots live in `inputs/screenshots/`, captured by the user. Never fabricate reviews. Keep the product original (no third-party IP).
- **Source of truth:** working `config.csv` + `beats.csv` in `inputs/` (exported from Google Sheets); the `example-*.csv` in the skill folder are templates only. Per-beat assets are named `beat-N-[label]-[type].ext`.

Start by running `/ugc-reaction-studio`.

---
> Source: [docusphere/ugc-reaction-studio](https://github.com/docusphere/ugc-reaction-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
