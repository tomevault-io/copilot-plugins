## resolve-ai-toolkit

> AI-powered MCP toolkit for DaVinci Resolve — Fusion/VFX, editing, audio, color grading, and organization.

# Resolve AI Toolkit

AI-powered MCP toolkit for DaVinci Resolve — Fusion/VFX, editing, audio, color grading, and organization.

**21 MCP tools** across 5 modules. Connects directly to Resolve via the Scripting API.

## Quick Start

```bash
# Install
cd /Volumes/IF\ 4TB\ NVME_9100PRO/Coding/resolve-ai-toolkit
uv venv --python 3.12 && source .venv/bin/activate && uv pip install -e .

# Run as MCP server
python -m resolve_toolkit
```

**Claude Code MCP config** (`~/.claude/settings.json`):
```json
"resolve-ai-toolkit": {
  "command": "/Volumes/IF 4TB NVME_9100PRO/Coding/resolve-ai-toolkit/.venv/bin/python",
  "args": ["-m", "resolve_toolkit"]
}
```

**Requires:** DaVinci Resolve Studio running, external scripting enabled.

## Module Reference

### Fusion/VFX (5 tools)

| Tool | Key Actions | Purpose |
|------|-------------|---------|
| `fusion_comp` | list_comps, inspect, create, import_comp, export_comp, delete | Manage Fusion compositions on timeline items |
| `fusion_tools` | list_tools, get_input, set_input, get_all_inputs, set_passthrough | Manipulate Fusion nodes — read/write any input |
| `fusion_text` | scan_text, set_text, set_font, set_color | TextPlus field operations across generators |
| `fusion_effects` | insert_generator, insert_title, insert_composition | Insert Fusion generators/titles into timeline |
| `fusion_recipes` | list_recipes, build, build_lower_third, build_title_card | AI-guided comp building from recipes |

**Recipes:** lower_third, title_card, name_super, text_overlay, chroma_key, luma_key, blend_layers, solid_background, vignette, blur_transition

### Audio/Fairlight (4 tools)

| Tool | Key Actions | Purpose |
|------|-------------|---------|
| `audio_manage` | list_tracks, insert_audio, get/set_voice_isolation, add_track | Audio track management |
| `audio_presets` | list_presets, apply_preset, scan_audio_tracks | Fairlight preset management |
| `audio_transcribe` | create_subtitles, list_subtitle_tracks, export_subtitles | Transcription and subtitles |
| `audio_analyze` | find_gaps, coverage_report, suggest_music_placement | Audio analysis and placement |

### Editing (5 tools)

| Tool | Key Actions | Purpose |
|------|-------------|---------|
| `edit_timeline` | analyze, detect_scene_cuts, duplicate, export | Timeline-level operations |
| `edit_clips` | list_clips, get/set_property, create_compound, create_fusion_clip | Clip management |
| `edit_pacing` | analyze, suggest_trims, analyze_transitions | Pacing analysis with style targets |
| `edit_markers` | list, add, delete, bulk_add, export | Marker management |
| `edit_assembly` | build_selects, reorder_by_timecode, find_gaps | Rough cut helpers |

**Pacing styles:** documentary, commercial, narrative, music_video, corporate, social_media

### Color (4 tools)

| Tool | Key Actions | Purpose |
|------|-------------|---------|
| `look_develop` | generate, apply, list_presets, apply_preset | Natural-language look development |
| `shot_match` | analyze_hero, copy_to_all, apply_cdl_to_track | Shot matching from hero |
| `grade_analyze` | describe, compare, scan_track | Grade analysis in plain language |
| `mood_grade` | translate, apply, blend, reset | Emotional language to CDL |

**30 presets** in 5 categories: film_stocks, moods, genres, decades, emotions

### Organization (3 tools)

| Tool | Key Actions | Purpose |
|------|-------------|---------|
| `organize_clips` | scan_metadata, group_by, auto_color_code | Smart clip grouping |
| `organize_bins` | list_folders, create_folder, move_clips, auto_organize | Media Pool bin management |
| `organize_metadata` | batch_read, batch_write, copy_metadata | Metadata batch operations |

## Architecture

```
resolve_toolkit/
├── server.py          # Main MCP server — registers all 21 tools
├── connection.py      # Shared Resolve connection + helpers (check, get_timeline, get_item, etc.)
├── fusion/tools.py    # 5 Fusion/VFX tools
├── fusion/templates.py # Recipe builders (lower thirds, keys, backgrounds, etc.)
├── audio/tools.py     # 4 Audio/Fairlight tools
├── editing/tools.py   # 5 Editing tools
├── editing/pacing.py  # Pacing analysis engine
├── color/tools.py     # 4 Color tools
├── color/engine.py    # CDL math, mood-to-CDL translation
├── color/presets.py   # 30 cinematic look presets
├── organize/tools.py  # 3 Organization tools
```

All tools use `action` dispatch: `tool(action="action_name", params={...})`.

All tools share `connection.py` for Resolve access — lazy connection on first call.

## Resolve API Gotchas

- `GetItemListInTrack()` returns **None** for empty tracks, not `[]`
- `GetLeftOffset()` returns **None** for transitions/generators (type detector)
- Fusion comp indices are **1-based**, not 0-based
- `CopyGrades()` requires Color page active
- Timeline handles go stale after `SetCurrentTimeline()` — reacquire
- Always `time.sleep(0.2-0.5)` after Fusion input writes

---
> Source: [allwavemedia/resolve-ai-toolkit](https://github.com/allwavemedia/resolve-ai-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
