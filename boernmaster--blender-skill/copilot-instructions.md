## blender-skill

> This project sets up `blender-remote` on a Linux machine and connects it to Claude Code via MCP, enabling natural language control of Blender.

# Blender Remote MCP — Claude Code Context

This project sets up `blender-remote` on a Linux machine and connects it to Claude Code via MCP, enabling natural language control of Blender.

## Environment

- Remote Linux machine with NVIDIA GPU + CUDA
- Python dependencies managed via `uv` (see `pyproject.toml`)
- Blender runs headless on port **6688**
- Claude Code connects to Blender via the `blender` MCP server

## Starting a Session

Two terminals required on the same machine:

**Terminal 1 — Blender:**
```bash
blender-start   # or: blender-remote-cli start --background --port 6688
```

**Terminal 2 — Claude Code:**
```bash
claude
```

Then verify MCP is connected: `/mcp` — you should see `blender` listed.

## Key Commands

| Task | Command |
|---|---|
| Start Blender | `blender-start` |
| Stop Blender | `blender-stop` |
| Restart Blender | `blender-restart` |
| Check MCP servers | `claude mcp list` |
| Kill port manually | `fuser -k 6688/tcp` |

## MCP Server Registration

```bash
claude mcp add blender \
  -e BLENDER_HOST=localhost \
  -e BLENDER_PORT=6688 \
  -- uvx blender-remote --host localhost --port 6688
```

## Common Issues

- **Port in use:** `fuser -k 6688/tcp`
- **CUDA not detected:** check `nvidia-smi`, verify the startup script at `~/.config/blender/<version>/scripts/startup/enable_cuda.py`
- **Missing `requests` module:** run `subprocess.call([sys.executable, '-m', 'pip', 'install', 'requests'])` in Blender's Python console
- **`blender-remote-cli init` fails:** use `blender-remote-cli init --blender-path $(which blender)`

## Output Paths

| Type | Path |
|---|---|
| Render frames | `output/frames/` |
| Final renders / GIFs | `output/` |
| Blender scene file | `work/scene.blend` |

`blender-start` must be run from this project directory so that `os.getcwd()` in render scripts resolves to the correct location.

## Notes

- Both Blender and Claude Code must run simultaneously — if Blender Remote stops, the MCP connection is lost
- CUDA is enabled via a startup script so it persists across sessions
- The `.venv` is managed by `uv`; use `uv add <package>` to add dependencies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boernmaster)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/boernmaster)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
