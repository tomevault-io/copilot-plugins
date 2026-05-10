## resolve-mcp-server

> You are controlling DaVinci Resolve via the MCP tools registered by this server.

# DaVinci Resolve MCP Server

You are controlling DaVinci Resolve via the MCP tools registered by this server.

## Project Status

**Built and verified working.** 53 tools across 11 categories, plus Moondream AI vision.

### What's been tested end-to-end:
- Server starts in stdio mode (for Claude Desktop) and HTTP mode (for remote/mobile)
- All 53 tools register successfully
- Resolve connection works — reads product name, version, current page, project name
- Project listing and loading works (tested with "tutorial - clawdbot")
- Timeline listing works (9 timelines found with track counts)
- Frame export works — `ExportCurrentFrameAsStill()` from the Color page
- Moondream vision works — frame exported as 6MB PNG, converted to 253KB JPEG via Pillow, sent to Moondream API, got back detailed scene description
- **Clip transform** — zoom punch-in (1.2x) on interview clip, verified and reset
- **Markers** — added colored markers with AI-generated descriptions at specific timecodes
- **Clip replacement** — three-point overwrite edit: deleted b-roll clip on V2 and inserted replacement at same record position (AOA_CLIP_04 swapped with SOUTH_POLE_TAKE_OFF)
- **Clip deletion** — delete with optional ripple, returns frame range info
- **YouTube export** — quick export to Desktop using YouTube 1080p preset (3.4MB output)
- **Render with optimized media** — fallback when full-res decode fails (`UseOptimizedMedia: True`)
- **Media pool browsing** — listed 28 clips with durations and metadata
- **Track inspection** — listed all clips on all tracks with frame ranges
- **HTTP transport** — running on port 3001 via Cloudflare tunnel at `resolve.cochran.cloud`

### What still needs real-world testing:
- Color grading tools (LUT application, color versions)
- Title tools (Fusion Text+ insertion)
- Fusion tools (comp access)
- Media import tools
- Speed change tool
- Compound clip creation

## Architecture

```
resolve-mcp-server/
├── src/
│   ├── server.py                    # FastMCP server, stdio + HTTP transport
│   ├── services/
│   │   ├── resolve_connection.py    # Resolve API connection, helpers
│   │   └── moondream.py             # Moondream vision API client
│   └── tools/                       # 11 tool modules, each exports register(mcp)
│       ├── connection.py            # 4 tools
│       ├── project.py               # 6 tools
│       ├── timeline.py              # 8 tools
│       ├── media.py                 # 5 tools
│       ├── editing.py               # 7 tools
│       ├── color.py                 # 6 tools
│       ├── markers.py               # 3 tools
│       ├── titles.py                # 2 tools
│       ├── render.py                # 6 tools
│       ├── fusion.py                # 3 tools
│       └── vision.py                # 3 tools
├── .env                             # MOONDREAM_API_KEY (do NOT commit)
├── .env.example
├── .venv/                           # Python 3.14 virtualenv (Homebrew)
├── requirements.txt                 # mcp[cli], httpx, Pillow
├── start.sh                         # Launcher for Claude Desktop
├── CLAUDE.md
└── README.md
```

## Quick Reference

- **Always check status first** with `resolve_get_status` to see what project/timeline is active
- **Switch pages** before page-specific operations: `resolve_open_page("color")` before applying LUTs or exporting frames
- **Refresh LUTs** before applying: the `resolve_apply_lut` tool handles this automatically
- **mediaType=1 for b-roll**: When appending clips as b-roll, use `media_type=1` to add video only (preserves interview audio)
- **Timeline start frame is typically 86400** (01:00:00:00 at 24fps)
- **Frame export requires Color or Edit page** — switch with `resolve_open_page("color")` first

## Tool Categories

| Category | Tools | What they do |
|----------|-------|-------------|
| Connection | 4 | Status, page navigation, reconnect |
| Project | 6 | List/load/save/create projects, settings |
| Timeline | 8 | List/switch timelines, playhead, tracks, create |
| Media | 5 | List clips, import, create bins, append to timeline |
| Editing | 7 | Transform, speed, enable/disable, compound clips, delete, replace |
| Color | 6 | LUTs, color versions, export grades |
| Markers | 3 | Add/get/delete markers |
| Titles | 2 | Insert Fusion titles, modify text |
| Render | 6 | Quick export, render jobs, timeline export |
| Fusion | 3 | Fusion comp and tool management |
| Vision | 3 | AI frame analysis (Moondream) |

## Common Workflows

### "Apply a film look to all clips"
1. `resolve_open_page("color")`
2. For each clip, navigate playhead and `resolve_apply_lut("Film Looks/Rec709 Kodak 2383 D65.cube")`

### "Export for YouTube"
1. `resolve_quick_export("YouTube", output_dir="/path/to/output")`

### "What's in this shot?"
1. `resolve_open_page("color")` (needed for frame export)
2. `resolve_describe_frame()` — exports frame, converts to JPEG, sends to Moondream AI

### "Replace b-roll clip on V2"
1. `resolve_get_track_items("video", 2)` — find the clip to replace (note its clip_index)
2. `resolve_list_media()` — find the replacement clip name in the media pool
3. `resolve_replace_clip(track_index=2, clip_index=3, new_clip_name="NEW_CLIP.mov")` — replaces in place
   - Automatically matches the original clip's duration
   - Uses `media_type=1` (video only) by default to preserve audio on other tracks
   - Set `source_start_frame` to use a specific portion of the replacement clip

### "Delete a clip"
1. `resolve_delete_clip(track_type="video", track_index=2, clip_index=1)` — leaves a gap
2. `resolve_delete_clip(track_type="video", track_index=2, clip_index=1, ripple=True)` — closes the gap

### "Export for YouTube"
1. `resolve_quick_export("YouTube", output_dir="/Users/guycochranclawdbot/Desktop")`
2. If render fails with codec error, retry with optimized media enabled in project settings

### "Add lower third"
1. `resolve_insert_title("Speaker Name", font_size=0.04, position_x=0.5, position_y=0.15)`

### "AI-powered marker logging"
1. `resolve_open_page("color")` (needed for frame export)
2. `resolve_set_playhead(frame)` — navigate to the moment
3. `resolve_describe_frame()` — AI describes what's in the shot
4. `resolve_add_marker(frame, "Blue", "AI: Scene Description", note)` — log the description as a marker

## Running the Server

### Claude Desktop (stdio)
Add to Claude Desktop config:
```json
{
  "mcpServers": {
    "resolve": {
      "command": "/Users/guycochranclawdbot/resolve-mcp-server/start.sh"
    }
  }
}
```
The `.env` file is auto-loaded by the server (python-dotenv), so the Moondream API key is picked up automatically.

### Remote access (HTTP + Cloudflare tunnel)
```bash
cd ~/resolve-mcp-server
TRANSPORT=http PORT=3001 .venv/bin/python3 src/server.py
```

### Claude Code (direct testing)
```bash
cd ~/resolve-mcp-server
.venv/bin/python3 -c "
import sys, os
os.environ['RESOLVE_SCRIPT_API'] = '/Library/Application Support/Blackmagic Design/DaVinci Resolve/Developer/Scripting'
os.environ['RESOLVE_SCRIPT_LIB'] = '/Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Libraries/Fusion/fusionscript.so'
sys.path.insert(0, '/Library/Application Support/Blackmagic Design/DaVinci Resolve/Developer/Scripting/Modules/')
sys.path.insert(0, '.'
from src.services.resolve_connection import get_resolve
resolve = get_resolve()
print(resolve.GetProductName(), resolve.GetVersionString())
"
```

## Technical Notes

### Python environment
- **Venv**: `.venv/` created from Homebrew Python 3.14 (`/opt/homebrew/bin/python3`)
- System Python 3.9.6 is too old for the MCP SDK (requires 3.10+)
- PEP 668 requires venv for Homebrew Python — do NOT pip install globally

### FastMCP quirks
- Constructor accepts `host`, `port`, `json_response` — NOT `version`
- `run()` only accepts `transport` and `mount_path` — host/port go in constructor
- Transport values: `"stdio"`, `"sse"`, `"streamable-http"`

### Moondream vision pipeline
- Resolve exports frames as large PNGs (6MB+ for 1080p)
- `_prepare_image()` in moondream.py converts to JPEG, caps at 1920px wide (~253KB)
- Auth header: `X-Moondream-Auth: <raw_api_key>` (no Bearer prefix)
- API base: `https://api.moondream.ai/v1`
- Endpoints: `/caption`, `/detect`, `/query`, `/point`

### Resolve API constraints
- **Requires DaVinci Resolve Studio (paid)** — the scripting API is not available in the free version
- Resolve must be running on the same machine
- Frame export needs Color or Edit page active
- Cannot add transitions (use Cmd+T keyboard shortcut)
- Cannot trim or move clips directly — but CAN replace them (see below)
- No transport control (play/pause) — use keyboard shortcuts

### Clip replacement technique (key discovery)
The Resolve scripting API has no source viewer, overwrite edit, or three-point edit commands.
We worked around this by building a two-step "replace" operation:

1. **Delete** the old clip with `timeline.DeleteClips([clip], False)` (no ripple — keeps the gap)
2. **Insert** the new clip at the exact same record position using `pool.AppendToTimeline([clipInfo])`

The `clipInfo` dict format:
```python
{
    "mediaPoolItem": mp_item,     # MediaPoolItem object from pool search
    "trackIndex": 2,               # Video track number (1-based)
    "recordFrame": 86734,          # Timeline position (where to place it)
    "startFrame": 0,               # Source in point (within the new clip)
    "endFrame": 120,               # Source out point (auto-matched to original duration)
    "mediaType": 1,                # 1=video only, 2=audio only, omit=both
}
```

This is the `resolve_replace_clip` tool. It handles the full workflow: finds the old clip's position, searches the media pool for the replacement, calculates matching duration, deletes the old clip, and inserts the new one at the same spot.

### Render troubleshooting
- If a render fails with a codec/decode error, try enabling optimized media:
  `project.SetRenderSettings({'UseOptimizedMedia': True})`
- This is common when clips have codecs that can't decode at full resolution under render load

## Known Limitations

- Cannot add transitions directly (use Cmd+T via keyboard)
- Cannot trim or slide clips on the timeline — but CAN delete and replace them
- No transport control (play/pause) — use keyboard shortcuts
- Resolve must be running on the same machine as this server
- Frame export for vision requires being on Color or Edit page
- Clip replacement is a two-step delete+insert, so Cmd+Z will only undo the insert (not restore the delete)

---
> Source: [guycochran/resolve-mcp-server](https://github.com/guycochran/resolve-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
