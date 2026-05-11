## fl-studio-mcp

> **⚠️ macOS Primary, Windows Secondary** - This project primarily supports macOS with full auto-trigger functionality. Windows support is partially implemented but may require additional configuration.

# FL Studio Direct Integration - Documentation

## Platform Support

**⚠️ macOS Primary, Windows Secondary** - This project primarily supports macOS with full auto-trigger functionality. Windows support is partially implemented but may require additional configuration.

## Overview

This system allows Claude to interact with FL Studio's piano roll through direct file-based communication. It provides seamless integration between Claude's AI capabilities and FL Studio's music production environment with built-in timing delays to ensure reliable operation.

## Installation and Setup

### 1. Install MCP Server for Claude
```bash
./install_mcp_for_claude.sh
```
This registers the MCP server with Claude Code. The server is named `fl-studio-mcp`.

### 2. Usage Workflow

**For each session:**
1. **Open FL Studio** and open a piano roll
2. **Run ComposeWithLLM** once (Tools → Scripting → ComposeWithLLM) to initialize
3. **Start Claude** - the trigger mechanism is built into the MCP server
4. **No need to run separate scripts** - everything is handled automatically!

**Note:** Claude Code needs Accessibility permissions (System Settings → Privacy & Security → Accessibility) to send keystrokes to FL Studio.

## Getting Started (For AI Assistants)

If you're an LLM helping the user with this system, here's what you need to know:

### 1. Check Available Tools

First, verify you have access to these MCP tools (they'll have the `mcp__fl_studio_mcp__` prefix):
- `mcp__fl_studio_mcp__get_piano_roll_state` - Read the piano roll
- `mcp__fl_studio_mcp__send_notes` - Add or replace notes (including chords)
- `mcp__fl_studio_mcp__delete_notes` - Remove specific notes
- `mcp__fl_studio_mcp__clear_piano_roll` - Clear all notes
- `mcp__fl_studio_mcp__clear_queue` - Clear pending requests

If these aren't available, the MCP server needs to be started or reconnected. The server is registered as `fl-studio-mcp` in Claude Code.

### 2. Auto-Trigger System

The trigger mechanism is now **built into the MCP server**:
- When you send notes/requests, the MCP server automatically triggers FL Studio
- A 2-second delay ensures FL Studio has time to receive focus and process the request
- No separate auto-trigger script needed
- Works with both detached and docked piano roll windows

### 3. Always Start With State

Before making ANY changes, get the current piano roll state:

```python
state = mcp__fl_studio_mcp__get_piano_roll_state()
```

This tells you:
- What notes already exist
- The PPQ (timing resolution)
- What you're working with

### 4. Key Concepts for LLMs

**Built-in Auto-Trigger System:**
- MCP server writes requests to `mcp_request.json`
- MCP server automatically sends Cmd+Opt+Y (macOS) or Ctrl+Alt+Y (Windows)
- 2-second delay ensures proper focus handling
- FL Studio re-runs the last script (ComposeWithLLM)
- Notes appear automatically after the delay
- Cross-platform trigger support with focus management

**Time is Always in Quarter Notes:**
- `time=0` = beat 1
- `time=4` = beat 5 (measure 2 in 4/4)
- `duration=1` = quarter note
- `duration=4` = whole note
- Never use ticks - the bridge converts automatically

**Chords Are Just Multiple Notes:**
- A chord = multiple notes with the same `time` value
- Send them all in one `send_notes` call with matching times
- For example: C major = MIDI 60, 64, 67 all at same time

### 5. Common Interaction Patterns

**Pattern: Add a chord progression**
```python
# User: "Add a I-IV-V progression in C"

# Start by getting state (good practice)
state = mcp__fl_studio_mcp__get_piano_roll_state()

# Send each chord
mcp__fl_studio_mcp__send_notes([
    {"midi": 60, "duration": 4, "time": 0},
    {"midi": 64, "duration": 4, "time": 0},
    {"midi": 67, "duration": 4, "time": 0}
], mode="add")  # C major

mcp__fl_studio_mcp__send_notes([
    {"midi": 65, "duration": 4, "time": 4},
    {"midi": 69, "duration": 4, "time": 4},
    {"midi": 72, "duration": 4, "time": 4}
], mode="add")  # F major

mcp__fl_studio_mcp__send_notes([
    {"midi": 67, "duration": 4, "time": 8},
    {"midi": 71, "duration": 4, "time": 8},
    {"midi": 74, "duration": 4, "time": 8}
], mode="add")  # G major

# Notes will appear automatically!
```

**Pattern: Modify existing notes**
```python
# User: "Change that G note to an A"

# Get state to see what's there
state = mcp__fl_studio_mcp__get_piano_roll_state()

# Find the G note in the state (look for number: 67)
# Delete it
mcp__fl_studio_mcp__delete_notes([{"midi": 67, "time": 0}])

# Add replacement
mcp__fl_studio_mcp__send_notes([{"midi": 69, "duration": 4, "time": 0}])

# Changes appear automatically!
```

**Pattern: Start fresh**
```python
# User: "Clear everything and give me a C major scale"

# Use replace mode to clear and add
mcp__fl_studio_mcp__send_notes([
    {"midi": 60, "duration": 0.5, "time": 0},   # C
    {"midi": 62, "duration": 0.5, "time": 0.5}, # D
    {"midi": 64, "duration": 0.5, "time": 1},   # E
    {"midi": 65, "duration": 0.5, "time": 1.5}, # F
    {"midi": 67, "duration": 0.5, "time": 2},   # G
    {"midi": 69, "duration": 0.5, "time": 2.5}, # A
    {"midi": 71, "duration": 0.5, "time": 3},   # B
    {"midi": 72, "duration": 0.5, "time": 3.5}  # C
], mode="replace")

# Automatically clears old notes and adds new scale!
```

**Pattern: User makes manual edits**
```python
# User manually adds notes in FL Studio

# User: "I just added some notes manually"

# Remind user to refresh state
# "Please press Cmd+Opt+Y (or Ctrl+Alt+Y) so I can see your changes"

# User presses the key combo

# Get updated state
state = mcp__fl_studio_mcp__get_piano_roll_state()

# Now you can see the manual edits
```

### 6. Important Rules for LLMs

**DO:**
- ✅ Always get state first with `get_piano_roll_state()`
- ✅ Always specify `time` for every note (don't rely on defaults)
- ✅ Use quarter notes for time and duration
- ✅ Tell user notes will appear automatically after a brief delay
- ✅ Use `mode="add"` by default (accumulate changes)
- ✅ Use `mode="replace"` when user wants to start fresh
- ✅ Remind user to press Cmd+Opt+Y or use menu after manual edits to refresh state

**DON'T:**
- ❌ Don't tell user to click buttons (there are none - it's automatic!)
- ❌ Don't use ticks/PPQ directly - always use quarter notes
- ❌ Don't forget to get state before making changes
- ❌ Don't assume you can see manual edits without state refresh
- ❌ Don't worry about starting separate scripts - it's all built-in

### 7. Troubleshooting for LLMs

**User says "Nothing happened"**
- Ask: "Did you run ComposeWithLLM in FL Studio first?"
- Ask: "Does Claude have Accessibility permissions?" (System Settings → Privacy & Security → Accessibility)
- Ask: "Is FL Studio running and is a piano roll open?"
- Suggest: "Try pressing Cmd+Opt+Y manually or use Tools → Scripting → ComposeWithLLM"

**User says "It replaced everything"**
- Check if you used `mode="replace"` accidentally
- Should use `mode="add"` for incremental changes

**User says "Times are wrong"**
- Verify you're using quarter notes, not ticks
- Remember: time=4 is beat 5, not beat 4 (counting starts at 0)

**User makes manual edits**
- Remind them: "Press Cmd+Opt+Y to refresh the state so I can see your changes"

### 8. Musical Knowledge Tips

When helping with music:
- **MIDI note numbers**: C4 (middle C) = 60, each semitone = +1
- **FL Studio Octave Display**: FL Studio displays notes ONE OCTAVE HIGHER than standard MIDI convention. When you send MIDI 60 (C4 in standard), FL Studio shows it as C5. Use this reference:
  - MIDI 60 = C4 (standard) = **C5 in FL Studio**
  - MIDI 69 = A4 (standard) = **A5 in FL Studio**
  - MIDI 72 = C5 (standard) = **C6 in FL Studio**
- **Common durations**: 0.25=16th, 0.5=8th, 1=quarter, 2=half, 4=whole
- **Chord voicings**: Consider inversions for smoother voice leading
- **Bass notes**: Typically 1-2 octaves below the chord (MIDI 36-48 range)
- **Time signatures**: In 4/4, measures are 4 beats (time increments of 4)

### 9. Example Session

```
User: "Initialize FL Studio"
[User runs ComposeWithLLM script]
[User starts auto-trigger: python fl_studio_auto_trigger.py]

User: "Add a sad chord progression"

You: Get state first
> mcp__fl-studio__get_piano_roll_state()

You: Send Am - F - C - G progression
> mcp__fl-studio__send_notes([...]) # Am at time 0
> mcp__fl-studio__send_notes([...]) # F at time 4
> mcp__fl-studio__send_notes([...]) # C at time 8
> mcp__fl-studio__send_notes([...]) # G at time 12

You: "I've added a sad Am-F-C-G progression. Notes should appear automatically!"

[Notes appear in FL Studio automatically! 🎵]

User: "Great! Add a bass line"

You: Send bass notes
> mcp__fl-studio__send_notes([...]) # Bass notes at appropriate times

You: "Added bass notes!"

[Bass notes appear automatically! 🎸]

User: "Perfect!"

[Session done - all changes are live]
```

## Architecture

The system consists of three main components:

1. **MCP Server** (`fl_studio_mcp_server.py`) - Provides tools for Claude to send musical requests and automatically triggers FL Studio
2. **FL Studio Bridge Script** (`ComposeWithLLM.pyscript`) - Runs inside FL Studio to process requests automatically
3. **JSON Communication Files** - Request queue and state files for communication

### Communication Flow

```
Claude → MCP Server → Request Queue (JSON) → MCP Server Sends Trigger
                                                      ↓
                                       Sends Cmd+Opt+Y (macOS) or Ctrl+Alt+Y (Windows)
                                                      ↓
                                           FL Studio Re-runs Script
                                                      ↓
                                           Notes Added to Piano Roll
                                                      ↓
                                           State Export (JSON)
```

## Workflow

1. **Run ComposeWithLLM** (Tools → Scripting → ComposeWithLLM) → Once per session
   - Exports current piano roll state to `piano_roll_state.json`
   - Clears request queue (sets to `[]`)
   - No dialog appears

2. **Claude sends requests** → MCP Server writes to request queue

3. **MCP Server triggers FL Studio** → Automatically sends Cmd+Opt+Y with built-in delay

4. **FL Studio re-runs script** → Processes queue and updates state

5. **Notes appear** → Visible in piano roll after delay

## Available MCP Tools

### `get_piano_roll_state()`
Read the current piano roll state exported by FL Studio.

**Returns:** JSON with PPQ, note count, and all notes with properties (number, time, length, velocity, etc.)

**Example:**
```python
state = get_piano_roll_state()
# See all notes currently in the piano roll
```

### `send_notes(notes, mode="add")`
Send arbitrary notes to the piano roll.

**Parameters:**
- `notes`: List of note dictionaries
  - `midi`: MIDI note number (0-127)
  - `duration`: Duration in quarter notes
  - `time`: Start time in quarter notes
  - `velocity`: Optional, 0.0-1.0 (default 0.8)
- `mode`: "add" or "replace"

**Example:**
```python
# Send a C major chord at beat 0
send_notes([
    {"midi": 60, "duration": 2, "time": 0},
    {"midi": 64, "duration": 2, "time": 0},
    {"midi": 67, "duration": 2, "time": 0}
])

# Send a melody
send_notes([
    {"midi": 60, "duration": 0.5, "time": 0},
    {"midi": 62, "duration": 0.5, "time": 0.5},
    {"midi": 64, "duration": 0.5, "time": 1},
    {"midi": 65, "duration": 0.5, "time": 1.5}
])
```


### `delete_notes(notes)`
Delete specific notes from the piano roll.

**Parameters:**
- `notes`: List of note dictionaries to delete
  - `midi`: MIDI note number
  - `time`: Start time in quarter notes

**Example:**
```python
# Delete G note at beat 0
delete_notes([{"midi": 67, "time": 0}])

# Delete multiple notes
delete_notes([
    {"midi": 67, "time": 0},
    {"midi": 72, "time": 4}
])
```

### `clear_queue()`
Clear all pending requests without affecting the piano roll.

**Use case:** Rarely needed in auto mode, but can discard pending requests.

**Example:**
```python
# Clear pending requests
clear_queue()
```

## Modes

### Add Mode (`mode="add"`)
- Appends to existing queue
- Adds to existing notes in piano roll
- Default behavior

### Replace Mode (`mode="replace"`)
- Clears queue first
- Adds `{"action": "clear"}` to delete all notes
- Then adds new notes
- Use for complete rewrites

## Time and Duration

- **Time units:** Quarter notes (beats)
  - `time=0` → Beat 1
  - `time=4` → Beat 5 (start of measure 2 in 4/4)
  - `time=8` → Beat 9 (start of measure 3)

- **Duration units:** Quarter note multipliers
  - `0.25` = 16th note
  - `0.5` = 8th note
  - `1.0` = Quarter note
  - `1.5` = Dotted quarter
  - `2.0` = Half note
  - `4.0` = Whole note

- **PPQ (Pulses Per Quarter):** Typically 96 or 480
  - Ticks = PPQ × Quarter Notes
  - Conversion handled automatically by bridge script

## Request Queue System

Requests accumulate as a list of actions in `mcp_request.json`:

```json
[
  {"action": "delete_notes", "notes": [{"midi": 67, "time": 0}]},
  {"action": "add_notes", "notes": [{"midi": 69, "duration": 2, "time": 0}]},
  {"action": "add_notes", "notes": [{"midi": 65, "duration": 2, "time": 2}]}
]
```

**Processing order:** Actions are executed in order when auto-trigger fires.

## File Locations

**FL Studio scripts directory:**
```
~/Documents/Image-Line/FL Studio/Settings/Piano roll scripts/
├── ComposeWithLLM.pyscript  (bridge script - auto mode)
├── mcp_request.json          (request queue)
├── mcp_response.json         (execution results)
└── piano_roll_state.json     (exported piano roll state)
```

**Source repository:**
```
/Users/calvinw/develop/fl-studio-mcp/
├── ComposeWithLLM.pyscript      (source bridge script)
├── fl_studio_mcp_server.py       (MCP server with built-in trigger)
├── install_prerequisites.sh     (install uv and Python)
├── install_mcp_for_claude.sh    (register MCP server)
└── CLAUDE.md                     (this file)
```

## Typical Workflows

### Building a Chord Progression

```python
# Initialize: Run ComposeWithLLM in FL Studio
# MCP server handles triggering automatically

# Send chord progression
send_notes([
    {"midi": 60, "duration": 4, "time": 0},
    {"midi": 64, "duration": 4, "time": 0},
    {"midi": 67, "duration": 4, "time": 0}
])  # C major

send_notes([
    {"midi": 65, "duration": 4, "time": 4},
    {"midi": 68, "duration": 4, "time": 4},
    {"midi": 72, "duration": 4, "time": 4}
])  # F minor

# Notes appear automatically!
```

### Modifying Existing Notes

```python
# Get current state
state = get_piano_roll_state()

# Delete a specific note
delete_notes([{"midi": 67, "time": 0}])

# Add replacement
send_notes([{"midi": 69, "duration": 4, "time": 0}])

# Changes appear automatically!
```

### Starting Fresh (Replace Mode)

```python
# Clear everything and add new progression
send_notes([
    {"midi": 60, "duration": 2, "time": 0},
    {"midi": 64, "duration": 2, "time": 0},
    {"midi": 67, "duration": 2, "time": 0}
], mode="replace")

# Old notes cleared, new chord appears!
```

## Tips

1. **Always specify `time`** - Every note should have an explicit time position
2. **Use quarter notes** - All time/duration values are in quarter note units
3. **Get state first** - Call `get_piano_roll_state()` before making changes
4. **Refresh after manual edits** - User should press Cmd+Opt+Y to update state
5. **Auto-trigger built into MCP server** - No separate scripts needed, everything is automatic

## Troubleshooting

**Changes not appearing?**
- Verify ComposeWithLLM was run once in FL Studio
- Check FL Studio window is active
- Ensure Claude Code has Accessibility permissions (System Settings → Privacy & Security → Accessibility)

**Notes at wrong positions?**
- Check PPQ value in state export
- Ensure time values are in quarter notes, not ticks

**Trigger not working?**
- Try pressing Cmd+Opt+Y manually or use Tools → Scripting → ComposeWithLLM
- Run ComposeWithLLM in FL Studio again
- Restart Claude Code to reconnect the MCP server

**Script not responding?**
- Check that ComposeWithLLM.pyscript is copied to Piano roll scripts directory
- Verify MCP server is connected in Claude Code
- Restart Claude Code after code changes

## Future Enhancements

Potential additions:
- Delete by time range
- Modify note properties (velocity, length)
- Pattern generation tools
- Harmonic analysis tools
- MIDI file import/export
- Cross-platform UI for manual trigger

---
> Source: [calvinw/fl-studio-mcp](https://github.com/calvinw/fl-studio-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
