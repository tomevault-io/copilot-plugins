## blender-agent

> Drive Blender via HTTP POST to `localhost:5656`. See the `blender` skill for

# Blender Agent

Drive Blender via HTTP POST to `localhost:5656`. See the `blender` skill for
command syntax, response format, screenshots, logging, and common patterns.

## Project structure

```
blender_agent/__init__.py    # The Blender addon (HTTP server + exec engine)
blender_agent/blender_manifest.toml
start_server.py              # Launcher: connects or starts Blender, blocks until ready
output/                      # Render output (gitignored)
.claude/skills/              # Agent Skills (auto-loaded by Claude when relevant)
  blender/                   # General Blender automation (start here)
  blender-3d/                # 3D scenes, materials, animation, rendering
  blender-vse/               # Video Sequence Editor
  blender-geometry-nodes/    # Geometry Nodes (procedural geometry, instancing)
  blender-laser/             # Laser beams with raycast reflection
  blender-projector/         # Spotlight projection patterns (grid, dots, radial, circles)
  blender-audioviz/          # Audio visualization (extraction, patterns, lighting rigs)
```

## Reference code

`../blender-mcp` contains a Blender MCP integration (by ahujasid). Useful as reference for:
- Blender Python API patterns (addon.py ~2600 lines)
- MCP server implementation (src/blender_mcp/server.py)

## Blender version

Target **Blender 5.1+** only. No backwards compatibility needed.

The skills in `.claude/skills/` are the source of truth for Blender 5.1 API usage. Always
follow the patterns in skills rather than relying on training data, which is mostly pre-5.0.
When you hit an API error, fix it and update the relevant skill so it stays accurate.

**Do not use auto-memory at all.** Do not create or write to MEMORY.md or any files in the
memory directory. All project knowledge belongs in CLAUDE.md and the skill files in
`.claude/skills/`, which ship with the project.

If stuck, search the web. Key docs:
- Python API: https://docs.blender.org/api/5.1/
- Release notes (API changes): https://developer.blender.org/docs/release_notes/5.1/python_api/
- VSE changes: https://developer.blender.org/docs/release_notes/5.1/sequencer/

## Skill usability test

When asked to test a skill (e.g. "test the geometry nodes skill"), launch a background sub-agent
that builds something non-trivial using only the skill documentation. This validates that the
skills are accurate, complete, and usable by an agent without prior knowledge.

### How to run

1. **Pick a test project** appropriate to the skill being tested. It should exercise multiple
   sections of the skill (not just the basics). Example test projects:
   - `blender-3d`: animated scene with materials, lighting, camera, and rendered output
   - `blender-vse`: multi-strip timeline with text, transitions, and video render
   - `blender-geometry-nodes`: procedural scene with instancing, keyframed inputs, and render
   - `blender-projector`: multi-pattern spotlight scene with fog, animated params, and render

2. **Launch a background sub-agent** (Task tool, `run_in_background: true`) with instructions to:
   - Read the relevant skill files from `.claude/skills/`
   - Verify Blender is running
   - Build the project step by step, checking for errors after each command
   - Render a test frame and report the path
   - Report: what it built, any skill docs that were confusing/wrong/missing, errors hit and how resolved

3. **Monitor progress** by tailing the agent's output file (returned in the Task result).
   Check in at milestones if the user wants visibility.

4. **Review the result**: inspect the rendered output, then fix any skill documentation bugs
   the sub-agent identified. Commit fixes if warranted.

### What the sub-agent should NOT do

- Modify any project files (skills, CLAUDE.md, code) — only send commands to Blender
- Use knowledge outside the skill files — the test validates the docs, not the agent's training data

## Known Blender 5.1 bugs

- **Strip modifiers crash**: `strip.modifiers.new()` segfaults. Avoid until fixed.
- **Render can crash**: threading segfault in `libIlmThread`. Use `resolution_percentage = 50` for test renders.
- **Never call `depsgraph.update()` in frame handlers**: Causes crashes (GIL contention →
  segfault) or severe playback stutter. For hiding objects from `ray_cast`, use
  `hide_viewport = True` without `depsgraph.update()` — it works. See the laser skill.

---
> Source: [ptrthomas/blender-agent](https://github.com/ptrthomas/blender-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
