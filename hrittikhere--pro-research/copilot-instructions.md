## pro-research

> Multi-agent research skill for Claude Code. Produces evidence-backed reports and blog posts with Excalidraw diagrams and Remotion animated GIFs.

# Pro-Research Orchestrator

Multi-agent research skill for Claude Code. Produces evidence-backed reports and blog posts with Excalidraw diagrams and Remotion animated GIFs.

## Usage

```
/pro-research <topic> [--goal "..."] [--audience "..."] [--output report|blog|both] [--no-confirm]
```

## Structure

- `.claude/commands/pro-research.md` -- orchestrator skill
- `remotion/` -- Remotion project for animated GIF visualizations
- `templates/` -- output templates and Excalidraw JSON skeletons

## Conventions

- When running Remotion renders, always `cd remotion && npm install` first if `node_modules/` is missing
- Excalidraw JSON must include required fields: `id`, `type`, `x`, `y`, `width`, `height`, `seed`
- Evidence cards must include `source_url` for citation
- Save checkpoints after evidence collection using `save_checkpoint`
- Remotion render command: `cd remotion && npx remotion render src/index.ts <CompositionId> out/<name>.gif --props='<json>' --codec=gif`

## Available Remotion Compositions

BarChart, Timeline, FlowDiagram, PieChart, ComparisonTable, StatCard, RadarChart

## Depth Presets

- `--depth quick`: 1-2 agents, 2-3 questions, 0-1 diagrams, 0 GIFs
- `--depth medium` (default): 2-3 agents, 3-5 questions, 1-2 diagrams, 1-2 GIFs
- `--depth deep`: 3 agents, 5-7 questions, 2-3 diagrams, 2-4 GIFs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hrittikhere) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
