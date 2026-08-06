## blender-claude-plugin

> This file provides guidance to AI coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## What This Is

A skill plugin containing 8 expert skills for Blender 5.x Python scripting (targeting 5.0 and 5.1). This is a content-only repo — no build system, no tests, no code to run. All files are Markdown and JSON.

## Architecture

This is a **skill plugin**. The plugin system works as follows:

- `.claude-plugin/plugin.json` — Plugin metadata (name, description, version). This is what the host agent reads to discover the plugin.
- `.claude-plugin/marketplace.json` — Marketplace listing for shared discovery.
- `skills/<skill-name>/SKILL.md` — The main skill file. Must have YAML frontmatter with `name` and `description` fields. The `description` is used to decide when to activate the skill.
- `skills/<skill-name>/references/*.md` — Supporting reference files that SKILL.md points to. These are read on-demand when the skill is active.
- `docs/blender-mcp-setup.md` — User-facing setup guide for the official Blender MCP Server. Skills link here from their MCP-First section.

## Blender MCP Server (target integration)

Skills target the **official Blender MCP Server** by Blender Lab — <https://www.blender.org/lab/mcp-server/>, source <https://projects.blender.org/lab/blender_mcp>.

Architecture:

```
Claude Code ──MCP (stdio/http)──► blender-mcp ──TCP localhost:9876──► Blender Add-on (bpy)
```

Public tools (use these names verbatim in skill content):

- Action: `execute_blender_code`
- Inspection: `get_objects_summary`, `get_object_detail_summary`, `get_blendfile_summary_datablocks`, `get_blendfile_summary_missing_files`, `get_blendfile_summary_of_linked_libraries`, `get_blendfile_summary_path_info`, `get_blendfile_summary_usage_guess`
- Docs: `search_api_docs`, `get_python_api_docs`, `search_manual_docs`
- UI: `get_screenshot_of_window_as_image`, `get_screenshot_of_window_as_json`, `get_screenshot_of_area_as_image`, `jump_to_tab_by_name`, `jump_to_tab_by_space_type`, `jump_to_view3d_object_by_name`, `jump_to_view3d_object_data_by_name`
- Render: `render_thumbnail_to_path`, `render_viewport_to_path`

Requires Blender 5.1+ (add-on uses 5.1 extension manifest). Do not reference legacy third-party `blender-mcp` (e.g. `ahujasid/blender-mcp`) — that integration was replaced.

## Skill Conventions

Each skill follows the same structure:

1. **SKILL.md** — YAML frontmatter (`name`, `description` with trigger phrases), then:
   - Overview section
   - MCP-First Approach (prefer official Blender MCP Server when available, list concrete tool names, fall back to Python scripts)
   - Task Decision Tree (maps user intents to sections/references)
   - Recipes/patterns for common tasks
   - Debugging & Optimization section
   - Pointers to reference files

2. **references/python_api.md** — Complete Python code templates with correct `bpy` type strings, ready to copy into scripts

3. **Domain-specific reference** — One of: `node_reference.md`, `constraint_reference.md`, `modifier_reference.md`, `physics_reference.md`, `data_model_reference.md`, or `settings_reference.md`

### SKILL.md Frontmatter Format

```yaml
---
name: blender-<domain>
description: This skill should be used when working with Blender <domain>... Triggers on "<keyword1>", "<keyword2>"... If a Blender MCP server is available, prefer using that for direct Blender interaction.
---
```

The `description` field drives skill activation. It must include concrete trigger phrases that match user prompts.

## Editing Guidelines

- All API content targets **Blender 5.x** (Python API via `bpy`). Mark any version-specific features (e.g., "new in 5.0", "new in 5.1").
- Node/modifier/constraint type strings must be exact (e.g., `GeometryNodeMeshCube`, `CompositorNodeDenoise`, `'INVERSE_KINEMATICS'`). Incorrect type strings break scripts silently.
- Reference tables must include the Python type string column — this is the primary lookup value.
- Code examples should be complete and runnable (imports, object selection, mode switching).
- Keep the MCP-first pattern: every SKILL.md must reference the official Blender MCP Server tool names (`execute_blender_code`, etc.) and link to `docs/blender-mcp-setup.md`.

## Git Conventions

Commits use gitmoji + conventional commits format:

```
🎉 feat: description
📝 docs: description
🐛 fix: description
♻️ refactor: description
```

---
> Source: [ra100/blender-claude-plugin](https://github.com/ra100/blender-claude-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
