## world-builder

> This repository turns one reference image from `input/` into a stylized Blender scene under `worlds/<slug>/`.

# world-builder - Codex Project Instructions

This repository turns one reference image from `input/` into a stylized Blender scene under `worlds/<slug>/`.

The Claude source setup remains useful, but Codex should use the migrated project surfaces:

- Project instructions: `AGENTS.md`
- Codex skills: `.agents/skills/*/SKILL.md`
- Codex MCP config: `.codex/config.toml`
- Canonical implementation scripts: `.claude/scripts/`
- Reference docs: `docs/ARCHITECTURE.md`, `docs/PIPELINE.md`, `docs/api-research.md`, `docs/style-references.md`

## Start Here

1. For any world-building request, read `.agents/skills/build-world/SKILL.md` first.
2. When that skill names another skill, read the matching file in `.agents/skills/<skill>/SKILL.md`.
3. Treat `.claude/scripts/` as the executable implementation layer. Do not rename or duplicate these scripts just because they live under `.claude/`.
4. If the migrated skill text says `$1`, read it as the world slug. If it says `$2`, read it as the object or material id for that skill.
5. Before improvising pipeline behavior, check `docs/PIPELINE.md` and the relevant skill.

## MCP And Tool Access

The project MCP config has been migrated to `.codex/config.toml`:

- `blender`: stdio server pointing at the local `blender-mcp` executable installed via `uv tool install blender-mcp` (path is host-specific — see `.codex/config.toml.example`)
- `fal-ai`: streamable HTTP server at `https://mcp.fal.ai/mcp`, using `FAL_KEY`

Before live Blender or fal work, verify the current Codex session actually exposes the tools. Use `tool_search` for Blender and fal if they are not already visible. The presence of `.codex/config.toml` only means the repo is configured; it does not prove this running session has loaded the MCP tools.

If Blender MCP tools are missing, stop before touching a live `.blend`. You may still inspect files, refine plans, edit scripts, prepare prompts, and run non-Blender checks.

If fal MCP tools are missing but `FAL_KEY` is available, prefer the local Node wrappers under `.claude/scripts/asset-pipeline/`. Load `.env` into the shell environment without printing secrets.

Use Codex tool equivalents:

- Claude `Read` for text -> shell read commands or normal file inspection.
- Claude `Read` for images -> `view_image` when given a local image path, or the available multimodal image input.
- Claude `Write` / `Edit` -> `apply_patch` for manual edits.
- Claude `Bash(...)` -> `functions.shell_command`.
- Claude `execute_blender_code` -> the actual Blender MCP tool, only when exposed in this session.

## Required Blender State

The user opens Blender. Do not launch Blender yourself for the world-building loop. If Blender is not open, or the MCP add-on is not running on port `9876`, stop and ask the user to open/fix it.

The first Blender action in a world build is saving the live scene as an absolute path:

`worlds/<slug>/world.blend`

After that:

- Live save: `bpy.ops.wm.save_mainfile()`
- Iteration snapshot: `bpy.ops.wm.save_as_mainfile(filepath=".../iterations/NNN.blend.bak", copy=True)`
- Final freeze: `bpy.ops.wm.save_as_mainfile(filepath=".../final.blend", copy=True)`, then `save_mainfile()`

Never call `save_as_mainfile` without `copy=True` after the initial pin, until the final flow explicitly requires it.

Set the viewport up immediately for recording: material preview, perspective view, Eevee Next, clean overlays, no grid clutter, no gizmos.

## World-Building Rules

- Generate control views before Blender placement: `controls/top.png`, `controls/fl45.png`, and `controls/fr45.png`.
- Use `controls/top.png` for placement coordinates. Back-project reviewed detection centroids through `plan.terrain.bbox`.
- Generate the whole landscape as one Tripo H3.1 textured mesh from the original 3/4 reference, not from flat `top.png`.
- Use PATINA only on flat primitive planes, especially water. Calibrate tile scale visually before continuing.
- Generate 3D meshes with Tripo H3.1 textured. Do not use Trellis fallback unless the project docs are changed.
- Set `rotation_mode = "XYZ"` on every Tripo-imported object before writing `rotation_euler`.
- Set object origin to bbox center-bottom before placement.
- Use Eevee Next for every loop render. Do not use Cycles in the iteration loop.
- Render the same three angles with `.claude/scripts/blender/render-3-views.py`.
- Iterate placement through the required refinement rounds before the toon pass.
- Apply `.claude/scripts/blender/apply-toon-shader.py` only after placement is final.
- Track fal spend in `worlds/<slug>/cost.json` and stop for confirmation before exceeding project caps.

Default caps:

- 25 total placed instances before asking.
- $20 fal spend per scene before asking.
- 25 loop iterations before escalating with the latest render and diagnosis.

## Output Layout

Every world uses:

```text
worlds/<slug>/
  source/reference.png
  plan.json
  controls/top.png
  controls/fl45.png
  controls/fr45.png
  assets/
  materials/
  iterations/
  world.blend
  final.blend
  final.render.png
  cost.json
```

Generated worlds are ignored by git except `worlds/_examples/`.

## Current Notes

The original Claude skills are preserved under `.claude/skills/`; the Codex-adapted copies live under `.agents/skills/`. If the two diverge, use the Codex copies for agent behavior and the `.claude/scripts/` files for execution.

All stored content and generated notes should be in English.

---
> Source: [Top3d-ai/world-builder](https://github.com/Top3d-ai/world-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
