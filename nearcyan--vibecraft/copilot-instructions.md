## vibecraft

> Manages subagent visualizations:

# Vibecraft - Technical Documentation

This document explains the Vibecraft codebase for future AI assistants working on this project.

## Project Purpose

Vibecraft visualizes Claude Code's activity in real-time as a 3D workshop. When Claude uses tools (Read, Edit, Bash, etc.), a character moves to corresponding workstations in a Three.js scene. The user can also send prompts to Claude from the browser via tmux integration.

## Architecture Overview

```
Claude Code → Hook Script → WebSocket Server → Browser (Three.js)
                  ↓
            events.jsonl (persistent log)
```

### Data Flow

1. **Claude Code** executes tools (Read, Edit, Bash, etc.)
2. **Hook script** (`hooks/vibecraft-hook.sh`) receives JSON via stdin from Claude Code's hook system
3. Hook writes to `~/.vibecraft/data/events.jsonl` AND POSTs to `http://localhost:4003/event`
4. **WebSocket server** (`server/index.ts`) broadcasts events to connected clients
5. **Browser** (`src/main.ts`) receives events and moves the Claude character

**Important:** Both hook and server use `~/.vibecraft/data/` as the data directory. This ensures they share the same files regardless of how vibecraft was installed (npx, global npm, or local dev).

### EventBus Architecture

Events are handled via a decoupled EventBus pattern, separating concerns into focused handler modules:

```
handleEvent(event)
    ↓
eventBus.emit(type, event, context)
    ↓
┌─────────────────────────────────────────────────────────┐
│  soundHandlers.ts      → Tool sounds, lifecycle sounds  │
│  notificationHandlers.ts → Zone floating text           │
│  characterHandlers.ts  → Movement, states               │
│  subagentHandlers.ts   → Task spawn/remove              │
│  zoneHandlers.ts       → Zone attention/status          │
│  feedHandlers.ts       → Thinking indicator             │
└─────────────────────────────────────────────────────────┘
    ↓
main.ts continues       → UI updates (DOM), special cases
```

**Files:**
- `src/events/EventBus.ts` - Core event dispatch with typed handlers
- `src/events/handlers/` - Handler modules (6 files)
- `src/events/handlers/index.ts` - Barrel export with `registerAllHandlers()`

**Adding a new event handler:**
```typescript
// In src/events/handlers/myHandlers.ts
import { eventBus } from '../EventBus'
import type { PreToolUseEvent } from '../../../shared/types'

export function registerMyHandlers(): void {
  eventBus.on('pre_tool_use', (event: PreToolUseEvent, ctx) => {
    if (!ctx.session) return
    // Handle the event...
  })
}

// Then add to src/events/handlers/index.ts:
import { registerMyHandlers } from './myHandlers'
export function registerAllHandlers(): void {
  // ... existing
  registerMyHandlers()
}
```

**EventContext** provides access to:
- `scene` - WorkshopScene for 3D updates
- `feedManager` - Activity feed
- `timelineManager` - Timeline strip
- `soundEnabled` - Sound toggle state
- `session` - Current session's claude, subagents, zone, stats

**Design principle:** EventBus handlers update 3D scene state. main.ts handles DOM UI updates (updateActivity, updateStats, etc.) and special cases (modals).

## Key Files

### `shared/types.ts`
Defines all TypeScript types used across server and client:

- `ClaudeEvent` - Union type of all event types (pre_tool_use, post_tool_use, stop, etc.)
- `TOOL_STATION_MAP` - Maps tool names to station names (Read→bookshelf, Bash→terminal)
- `StationType` - Valid station identifiers
- `ServerMessage` / `ClientMessage` - WebSocket protocol types

**Important**: When adding new tools, update `TOOL_STATION_MAP` to assign them to stations.

### `hooks/vibecraft-hook.sh`
Bash script that captures Claude Code events. The source lives in `hooks/vibecraft-hook.sh` but `npx vibecraft setup` copies it to `~/.vibecraft/hooks/vibecraft-hook.sh` (stable location).

**What it does:**
- Reads JSON from stdin (Claude Code pipes hook data)
- Transforms to our event format with `jq`
- Writes to `~/.vibecraft/data/events.jsonl` (append-only log)
- POSTs to server for real-time updates

**Cross-platform support:**
- Adds common tool paths to PATH (`/opt/homebrew/bin`, `/usr/local/bin`, etc.)
- Uses `find_tool()` function to locate `jq` and `curl` defensively
- Handles macOS timestamp differences (no `date +%N`)

**Known issue fixed**: Timestamp calculation used `$(date +%N)` which returns "087" etc. This was interpreted as octal. Fixed with `10#$ms_part` to force decimal.

**Compact JSON**: Must use `jq -n -c` (not just `jq -n`) to avoid multi-line output breaking JSONL format.

### Setup Process (`npx vibecraft setup`)

The setup command:
1. Copies `hooks/vibecraft-hook.sh` to `~/.vibecraft/hooks/vibecraft-hook.sh`
2. Creates `~/.vibecraft/data/` directory
3. Configures all 8 hooks in `~/.claude/settings.json`:
   - PreToolUse, PostToolUse, Stop, SubagentStop
   - SessionStart, SessionEnd, UserPromptSubmit, Notification
4. Backs up existing settings

**Why ~/.vibecraft/hooks/?** The hook path must be stable. If hooks pointed to the npm package location, they'd break when the package updates or npx cache clears.

### `server/index.ts`
Node.js WebSocket server:

- Watches `events.jsonl` with chokidar for file-based events
- Accepts POST `/event` for real-time hook notifications
- Broadcasts to WebSocket clients
- Calculates tool duration by matching pre/post events via `toolUseId`
- HTTP endpoints: `/health`, `/stats`, `/prompt`

**tmux integration**: POST to `/prompt` with `{prompt: "text", send: true}` runs:
```bash
tmux send-keys -t claude -l 'text' && sleep 0.1 && tmux send-keys -t claude Enter
```
The `-l` flag sends text literally, then Enter is sent separately after a delay.

### `src/scene/WorkshopScene.ts`
Three.js 3D scene setup:

- **World hex grid**: Subtle hex grid overlay across the entire floor, establishing hexagons as the "base reality" of the world
- **Hexagonal zones**: Each session gets a pointy-top hexagon platform that aligns with the world grid
- **Honeycomb layout**: Zones arranged in spiral pattern (center, ring 1, ring 2...)
- Defines 9 stations per zone with positions (see `STATION_POSITIONS` map)
- Performance optimizations applied:
  - `BasicShadowMap` instead of `PCFSoftShadowMap`
  - 512x512 shadow map (was 2048)
  - Single hemisphere light (removed 4 point lights)
  - Antialiasing disabled

**World grid**: `createWorldHexGrid()` renders ~127 hex outlines (6 rings from center) on the floor at y=0.01, using the same coordinate math as zone placement.

**Zone positioning**: Uses axial hex coordinates converted to cartesian. `indexToHexCoord()` maps linear index to spiral position.

### `src/entities/ClaudeMon.ts`
The main animated character (robot buddy). `Claude.ts` contains the legacy simpler character.

ClaudeMon features:

- States: `idle`, `walking`, `working`, `thinking`
- Color-coded ring: green=idle, blue=walking, orange=working, purple=thinking
- `moveTo(station)` - Animates movement using TWEEN.js
- `playRandomIdleBehavior()` - Triggers a random idle animation (used on zone focus)
- `playIdleBehavior(name)` - Play specific animation by name (dev panel)
- Uses `IdleBehaviorManager` for idle animations
- Uses `WorkingBehaviorManager` for station-specific work animations
- Accepts `ClaudeOptions` for customization: `scale`, `color`, `statusColor`, `startStation`

**State machine quirk**: Don't set idle state while walking, or the character stops mid-path. Check `state !== 'walking'` before setting idle.

### Animation System (`src/entities/`)

Modular animation architecture for character behaviors:

**Files:**
- `AnimationTypes.ts` - Shared interfaces, easing functions, utilities
- `IdleBehaviors.ts` - Random idle animations (fidgets, dances, emotes)
- `WorkingBehaviors.ts` - Station-specific work animations
- `animations/index.ts` - Barrel export for external imports

**Adding a new idle animation:**
```typescript
// In IdleBehaviors.ts
const myAnimation: IdleBehavior = {
  name: 'myAnimation',
  duration: 2,           // seconds
  weight: 5,             // higher = more likely to be picked
  categories: ['idle', 'emote'],
  update: (parts, progress, deltaTime) => {
    // progress goes 0→1 over duration
    parts.head.rotation.y = Math.sin(progress * Math.PI * 2) * 0.2
  },
  reset: (parts) => {
    parts.head.rotation.y = 0  // Always reset to avoid stuck poses!
  }
}

// Add to registry
export const IDLE_BEHAVIORS: IdleBehavior[] = [
  // ... existing
  myAnimation,
]
```

**Adding a new station animation:**
```typescript
// In WorkingBehaviors.ts
const myStationAnim: WorkingBehavior = {
  name: 'myStationAnim',
  loop: true,            // loops until state changes
  duration: 3,
  update: (parts, progress, deltaTime) => { /* ... */ },
  reset: (parts) => { /* ... */ }
}

// Map to station
export const STATION_ANIMATIONS: StationAnimations = {
  // ... existing
  myStation: myStationAnim,
}
```

**Available easing functions:** `easeInOut`, `easeOut`, `easeIn`, `bounce`, `elastic`, `easeOutBack`, `linear`

**Dev panel:** Press `Alt+D` to test animations in-browser.

### `src/entities/SubagentManager.ts`
Manages subagent visualizations:

- `spawn(toolUseId, description)` - Creates mini-Claude at portal when Task starts
- `remove(toolUseId)` - Removes subagent when Task completes
- Subagents are 60% scale, different colors (blue, emerald, pink, purple, amber, cyan)
- Positioned in a fan pattern around the portal to avoid overlap
- Tracks count for stats display

### `src/scene/ZoneNotifications.ts`
Floating notification system for zones:

- Shows tool completions as floating text above zones
- Auto-styling per tool type (icon + color)
- Stacking support (up to 5 notifications per zone)
- Float up and fade out animation

```typescript
// Show notification for a tool
scene.zoneNotifications.showForTool(sessionId, 'Read', 'config.ts')

// Show custom notification
scene.zoneNotifications.show(sessionId, {
  text: 'Custom message',
  style: 'success',  // success, info, warning, error, muted
  icon: '✨',
  duration: 3,
})
```

**Helper functions:**
- `formatFileChange(fileName, { added, removed, lines })` - "file.ts +5, -2"
- `formatCommandResult(command)` - Truncates long commands
- `formatSearchResult(pattern, matchCount)` - "pattern" → N matches

### `src/events/EventClient.ts`
WebSocket client for browser:

- Auto-reconnect with configurable interval
- `onEvent(handler)` - Called for each new event
- `onHistory(handler)` - Called with array of historical events (for batch processing)

**History handling**: History events arrive in chronological order (pre before post). The `onHistory` handler allows pre-scanning all events for `post_tool_use` to build a `completedToolUses` set BEFORE rendering icons. This prevents old pre_tool_use events from showing as "pending".

### `src/main.ts`
Main application entry point:

- Initializes scene, character, event client
- `handleEvent()` - Routes events to UI updates and character movement
- `addToTimeline()` - Adds emoji icons to the timeline strip
- `setupPromptForm()` - Handles browser prompt submission

**Tool movement logic**: Only moves Claude for tools that map to non-center stations. MCP browser tools (mcp__*) map to 'center' which would overwrite real movements.

### `src/ui/FeedManager.ts`
Activity feed panel manager:

- Renders events (prompts, tool uses, responses) in scrollable feed
- Session filtering via `setFilter(sessionId)`
- Auto-scroll with "jump to bottom" button
- Thinking indicator with animated dots

```typescript
feedManager.add(event, sessionColor)     // Add event to feed
feedManager.setFilter(sessionId)         // Filter by session
feedManager.showThinking(sessionId)      // Show "Claude is thinking..."
feedManager.hideThinking(sessionId)      // Remove thinking indicator
```

### `src/ui/QuestionModal.ts`
AskUserQuestion tool UI:

- Shows questions with option buttons
- Supports custom "Other" text input
- Sends responses via API to managed session or default tmux
- Updates zone attention state

### `src/ui/PermissionModal.ts`
Tool permission request UI:

- Shows when sessions run without `--dangerously-skip-permissions`
- Number key shortcuts (1-9) to select options
- No escape/click-outside close (user must choose)
- Manages zone attention and attention queue

### `src/ui/DrawMode.ts`
Hex painting mode for decorative coloring:

- `D` key toggles draw mode on/off
- 6 color palette (cyan, sky, blue, indigo, purple, teal)
- `1-6` to select colors, `0` for eraser
- `Q/E` to decrease/increase brush size (1-4 hexes radius)
- `R` to toggle 3D stacking mode (hexes grow when painted repeatedly)
- `X` to clear all painted hexes

```typescript
import { drawMode } from './ui/DrawMode'

drawMode.toggle()                    // Enter/exit draw mode
drawMode.getSelectedColor()          // Current color (or null if eraser)
drawMode.getBrushSize()              // 1-4
drawMode.is3DMode()                  // Whether stacking is enabled
drawMode.onChange((state) => {...})  // Subscribe to state changes
```

**Note:** Painted hexes are currently stored in memory only. See "Persistence" section for storage options.

### `src/ui/TextLabelModal.ts`
Custom modal for text tile input:

- Replaces browser's `prompt()` with themed textarea
- Multi-line support with word wrapping
- Character counter (500 char limit)
- Enter to save, Shift+Enter for newline, Escape to cancel

```typescript
import { showTextLabelModal } from './ui/TextLabelModal'

const text = await showTextLabelModal({
  title: 'Add Label',
  placeholder: 'Enter your text...',
  initialText: '',
  maxLength: 500,
})
```

Text tiles are rendered as 3D sprites with hex-styled beveled backgrounds.

### `index.html`
Single-page app with:

- HUD panels (connection status, current activity, stats)
- Icon timeline at bottom
- Prompt input form with "Send to tmux" toggle
- HTML structure only - CSS imported via main.ts

### `src/styles/`
CSS is organized into modular files:

| File | Purpose |
|------|---------|
| `index.css` | Entry point - imports all modules |
| `base.css` | Reset, body, layout, common animations |
| `sessions.css` | Session panel, session items, attention badges |
| `feed.css` | Activity feed, feed items, markdown styles |
| `prompt.css` | Prompt input, voice control, transcript |
| `hud.css` | Scene HUD, keybinds, timeline |
| `modals.css` | All modal styles (click menu, settings, questions, permissions) |

CSS is imported in `main.ts` via `import './styles/index.css'` and bundled by Vite.

## Event Types

| Type | When | Key Fields |
|------|------|------------|
| `pre_tool_use` | Before tool executes | `tool`, `toolUseId`, `input` |
| `post_tool_use` | After tool completes | `tool`, `toolUseId`, `success`, `duration` |
| `stop` | Claude stops responding | `reason` |
| `user_prompt_submit` | User sends prompt | `prompt` |
| `notification` | System notification | `message` |

## Station Mapping

```typescript
TOOL_STATION_MAP = {
  Read: 'bookshelf',
  Edit: 'workbench',
  Write: 'desk',
  Bash: 'terminal',
  Grep: 'scanner',
  Glob: 'scanner',
  WebFetch: 'antenna',
  WebSearch: 'antenna',
  Task: 'portal',
  TodoWrite: 'taskboard',
  // ... etc
}
```

## Common Tasks

### Adding a new station
1. Add position to `STATION_POSITIONS` in `WorkshopScene.ts`
2. Create station mesh in `createStations()`
3. Update `StationType` in `shared/types.ts`
4. Map relevant tools in `TOOL_STATION_MAP`

### Adding a new tool mapping
1. Edit `TOOL_STATION_MAP` in `shared/types.ts`
2. If new station needed, follow above

### Debugging events
1. Check `data/events.jsonl` for raw events
2. Enable debug: `VIBECRAFT_DEBUG=true npm run dev:server`
3. Browser console shows event client logs

### Performance issues
- Reduce shadow map size in `WorkshopScene.ts`
- Disable shadows entirely with `renderer.shadowMap.enabled = false`
- Reduce station geometry complexity

## Configuration

### Central Defaults

All default values are defined in **`shared/defaults.ts`** - the single source of truth:

```typescript
export const DEFAULTS = {
  SERVER_PORT: 4003,              // WebSocket/API server
  CLIENT_PORT: 4002,              // Vite dev server
  EVENTS_FILE: '~/.vibecraft/data/events.jsonl',
  SESSIONS_FILE: '~/.vibecraft/data/sessions.json',
  MAX_EVENTS: 1000,
  TMUX_SESSION: 'claude',
}
```

This file is imported by:
- `server/index.ts` - Server configuration (expands `~` at runtime)
- `vite.config.ts` - Dev server and build-time injection
- Frontend gets the port via Vite's `define` at build time

**Note**: The bash hook (`hooks/vibecraft-hook.sh`) also uses `~/.vibecraft/data/` by default, ensuring both hook and server share the same data directory.

### Data Directory

Vibecraft stores all data in `~/.vibecraft/data/`:
- `events.jsonl` - Event log (append-only)
- `sessions.json` - Session persistence
- `tiles.json` - Text tile labels
- `pending-prompt.txt` - Queued prompt (optional)

This location is used regardless of how vibecraft was installed (npx cache, global npm, or local dev), which fixes issues where hook and server ran from different installation directories.

## Persistence Architecture

See **[docs/STORAGE.md](docs/STORAGE.md)** for complete documentation of:
- localStorage keys and formats
- Server files and schemas
- Decision guide for choosing storage location
- Future account system migration path

**Quick summary:**

| Storage | Used For | Examples |
|---------|----------|----------|
| localStorage | User preferences, offline content | Volume, keybinds, hex art |
| Server files (`~/.vibecraft/data/`) | Shared state, server-managed data | Sessions, text tiles, events |

### Environment Variables

Environment variables override the defaults:

| Variable | Default | Purpose |
|----------|---------|---------|
| `VIBECRAFT_PORT` | 4003 | WebSocket/API server port |
| `VIBECRAFT_CLIENT_PORT` | 4002 | Vite dev server port |
| `VIBECRAFT_EVENTS_FILE` | ~/.vibecraft/data/events.jsonl | Event log path |
| `VIBECRAFT_DEBUG` | false | Enable verbose logging |
| `VIBECRAFT_TMUX_SESSION` | claude | tmux session for prompt injection |
| `VIBECRAFT_SESSIONS_FILE` | ~/.vibecraft/data/sessions.json | Session persistence file |
| `VIBECRAFT_DATA_DIR` | ~/.vibecraft/data | Hook data directory |
| `DEEPGRAM_API_KEY` | (none) | Deepgram API key for voice input |

A `.env` file is included with defaults - just run `npm run dev`.

## Bugs Fixed (Reference)

1. **Octal timestamp bug**: `087` milliseconds parsed as octal. Fix: `10#$ms_part`
2. **Multi-line JSON**: `jq -n` outputs pretty JSON. Fix: `jq -n -c` for compact
3. **Performance lag**: Too many lights + large shadows. Fix: Simplified lighting
4. **MCP tool movement**: Browser automation tools mapped to 'center', overwriting real movements. Fix: Skip moveTo for center-mapped tools
5. **History pending icons**: pre_tool_use showed as pending even when completed. Fix: Pre-scan history for post_tool_use before rendering
6. **tmux Enter key**: `C-m` and `Enter` inserted newlines in Claude's input. Fix: Send text with `-l`, then `Enter` separately after delay

## State Management

The system maintains state synchronization between:
- **Server** (`server/index.ts`) - Source of truth for managed sessions
- **Client** (`src/main.ts`) - Receives session updates via WebSocket

### Session Status Transitions

| Event | Status Change |
|-------|---------------|
| `user_prompt_submit` | → `working` (Claude is processing) |
| `pre_tool_use` | → `working` (tool running) |
| `post_tool_use` | Stay `working` (more tools may follow) |
| `stop` / `session_end` | → `idle` (Claude finished) |
| tmux session dies | → `offline` (health check) |
| No activity for 2 min | → `idle` (timeout failsafe) |

### Persistence

Sessions are persisted to `data/sessions.json`:
- Saved on: create, update, delete, link, status change
- Loaded on: server restart (sessions start as `offline`, health check updates)
- Contains: `sessions[]`, `claudeToManagedMap`, `sessionCounter`

### Reconciliation

Client rebuilds its local `claudeToManagedLink` map from server data on every `sessions` update. Server is authoritative for session linking.

### Health Checks

- **Session health** (every 5s): Runs `tmux list-sessions`, marks missing sessions `offline`
- **Working timeout** (every 10s): Sessions stuck `working` for >2 min auto-transition to `idle`

## Recent Features Added

- **Floating context labels**: Text sprites above stations showing current file/command
- **Thought bubbles**: Animated bubbles when Claude is thinking (full) or working (small)
- **Response capture**: Stop hook reads transcript to extract Claude's text response
- **Improved stations**: Desk (paper/pencil), Workbench (tools/gears), Taskboard (sticky notes)
- **Cancel button**: Sends Ctrl+C to tmux session via `/cancel` endpoint
- **Auto-expand textarea**: Input grows as you type, up to 200px max
- **Enter to send**: Enter submits, Ctrl+Enter for newline
- **Timeline deduplication**: Prevents duplicate icons on reconnect
- **Session orchestration**: Spawn and manage multiple Claude instances
- **New session modal**: Custom UI with name, directory, and command flags (-r, --chrome, --dangerously-skip-permissions)
- **Auto-linking**: Managed sessions automatically link to their 3D zones
- **Zone labels**: 3D floating labels update to show managed session names
- **Configurable ports**: Environment variables for client and server ports
- **Sound effects**: Synthesized audio feedback using Tone.js
- **Session persistence**: Sessions survive server restarts via `data/sessions.json`
- **Working timeout**: Sessions auto-transition to idle after 2 min without activity
- **State reconciliation**: Client rebuilds local state from server on reconnect
- **Hexagonal zones**: Bestagon platforms with honeycomb spiral layout
- **Click-to-create**: Click empty floor space to open new session modal
- **Session restart**: Offline sessions show 🔄 button to respawn tmux
- **Session refresh**: Manual 🔄 button triggers health check on all sessions
- **Offline styling**: Red highlight and border for offline sessions
- **Session tooltips**: Hover for tmux name, Claude ID, directory, last activity
- **Bidirectional selection**: Clicking sidebar ↔ 3D zone stays in sync
- **Zone floor status colors**: Zone floors glow based on status (idle=subtle zone color, working=cyan, waiting=amber, attention=red, offline=dim)
- **Attention system**: Zones pulse when sessions need attention (questions, finished)
- **Voice input**: Deepgram-powered speech-to-text with real-time transcription
- **Settings modal**: Consolidated settings for sound, tmux, and session management
- **Tab/Esc focus switching**: Quick toggle between Workshop (3D) and Activity Feed
- **Alt+Space shortcut**: Expand most recent "show more" in activity feed
- **World hex grid**: Visible hex grid across the floor, establishing hexagons as the "base reality"
- **Modular CSS**: Extracted inline styles to `src/styles/` directory (6 files)
- **Zone notifications**: Floating tool completion notifications above zones (file changes, commands, etc.)
- **Thinking indicator**: "Claude is thinking..." with animated dots in activity feed
- **Random idle on focus**: Claude plays a random idle animation when zone becomes active
- **Hex grid hover sounds**: Subtle audio feedback with distance-based pitch when hovering grid
- **ClaudeMon robot character**: Modular robot buddy with expressive animations
- **EventBus architecture**: Decoupled event handling with 6 focused handler modules
- **Extracted modals**: QuestionModal and PermissionModal as separate UI modules
- **Draw mode**: Hex painting with 6 colors, brush sizes, 3D stacking, and eraser
- **Zone elevation**: Painted hexes can stack in 3D when same color is painted repeatedly
- **Solid zone sides**: Raised zones show colored side faces, not just edge lines
- **Text label modal**: Custom themed textarea replaces browser prompt() for tile labels
- **Multi-line text tiles**: Word wrapping and hex-styled beveled backgrounds
- **Voice transcript streaming**: Real-time transcription streams to prompt input
- **Hosted voice mode**: Cloud proxy support for vibecraft.sh deployment
- **Spatial audio**: Distance-based volume and stereo panning for zone-aware sound effects
- **Context-aware Ctrl+C**: Copy when text selected, interrupt working session otherwise
- **Toast notifications**: Reusable toast system for user feedback (info, success, warning, error)
- **Zone Info modal**: Right-click zone to see session details, git status, tokens, files touched
- **Context-aware animations**: Claude reacts to events (victoryDance on git commit, headShake on errors, etc.)
- **Station panels**: Toggle with P key to see recent tool history per workstation (last 3 items)
- **Station glow pulse**: Brief ring highlight when tools use stations
- **Zone Command modal**: Right-click zone → C for quick prompt input positioned near the 3D zone

## Sound System

See **[docs/SOUND.md](docs/SOUND.md)** for complete documentation including the full sound catalog, spatial audio system, and implementation guide.

### Architecture

Sound effects are generated programmatically using **Tone.js** (Web Audio synthesis). No audio files needed.

**File:** `src/audio/SoundManager.ts`

```typescript
import { soundManager } from './audio/SoundManager'

// Initialize (must be called from user gesture)
await soundManager.init()

// Play sounds
soundManager.play('bash')           // Play by sound name
soundManager.playTool('Read')       // Play by tool name
soundManager.playResult(true)       // Play success/error

// Control
soundManager.setVolume(0.5)         // 0-1
soundManager.setEnabled(false)      // Mute all
```

### Sound Catalog

**Tools (8):**
| Sound | Trigger | Description |
|-------|---------|-------------|
| `read` | Read tool | Two-tone sine (A4→C5) |
| `write` | Write tool | Triple square blip (E5, E5, G5) |
| `edit` | Edit tool | Double triangle tap (E4→G4) |
| `bash` | Bash tool | DataBurst - 5 rapid sawtooth blips (C5) |
| `grep` | Grep/Glob | Soft sine + triangle (D4, F4) |
| `webfetch` | WebFetch/WebSearch | Ascending arpeggio (C5→E5→G5→C6) |
| `task` | Task tool | FM synth sweep (C3→C4) |
| `todo` | TodoWrite | Triple square checklist (E4, E4, G4) |

**States (3):**
| Sound | Trigger | Description |
|-------|---------|-------------|
| `success` | post_tool_use (success=true) | Rising chime (C5→G5) |
| `error` | post_tool_use (success=false) | Descending buzz (A2→F2) |
| `walking` | Claude moves | Soft double footstep (D4, D4) |

**Events (6):**
| Sound | Trigger | Description |
|-------|---------|-------------|
| `spawn` | Subagent created | Ethereal rise (C4→G5) |
| `despawn` | Subagent removed | Ethereal fall (G4→C3) |
| `prompt` | user_prompt_submit | Gentle acknowledgment (G4→D5) |
| `stop` | stop event | Completion chord (E4→G4→C5) |
| `notification` | notification event | Double ping (A4, A4) |
| `thinking` | Claude thinking state | Ambient two-tone (D4, F4) |

**Special Commands (1):**
| Sound | Trigger | Description |
|-------|---------|-------------|
| `git_commit` | Bash `git commit` | Satisfying fanfare (G3→B3→D4→G4 + shimmer) |

### Design Notes

- **Theme:** "Digital" - clean synth tones, quick response, non-intrusive
- **No audio files:** All sounds synthesized via Web Audio API
- **User gesture required:** `soundManager.init()` must be called from click/keypress
- **Aliases:** Glob→grep, WebSearch→webfetch, NotebookEdit→write

### Integration Points

Sounds are triggered via **EventBus handlers** in `src/events/handlers/soundHandlers.ts`:
- `pre_tool_use` → `soundManager.playTool(event.tool)`
- `pre_tool_use` (Bash with `git commit`) → `soundManager.play('git_commit')`
- `post_tool_use` → `soundManager.playResult(event.success)`
- `stop` → `soundManager.play('stop')`
- `user_prompt_submit` → `soundManager.play('prompt')`
- `notification` → `soundManager.play('notification')`
- Task tool → `spawn` / `despawn`

Character movement sounds (`walking`) are in `characterHandlers.ts`.

### Spatial Audio

Sounds can be positioned in 3D space based on zone location relative to the camera.

**Files:**
- `src/audio/SpatialAudioContext.ts` - Manages listener position and calculates spatial params
- `src/audio/SoundManager.ts` - Integrates spatial audio with sound playback

**Spatial modes:**
- `positional` - Affected by distance/pan (tool sounds, zone events)
- `global` - Always centered, full volume (git commit, notifications, UI sounds)

**Features:**
- **Distance volume:** Far zones are quieter (min 30% volume, never silent)
- **Stereo panning:** Zones to left/right of camera pan accordingly (±0.7 max)
- **Focus boost:** Selected zone gets +25% volume regardless of distance
- **Toggle in settings:** Users can disable spatial audio entirely

**Usage:**
```typescript
// Play with spatial positioning (for positional sounds)
soundManager.play('read', { zoneId: 'session-123' })
soundManager.playTool('Bash', { zoneId: 'session-123' })

// Play without spatial (for global sounds, or when no zone context)
soundManager.play('git_commit')  // git_commit is global by definition
soundManager.play('notification')  // notification is global
```

**Configuration (in main.ts):**
```typescript
// Set up zone position resolver
soundManager.setZonePositionResolver((zoneId) => scene.getZoneWorldPosition(zoneId))
soundManager.setFocusedZoneResolver(() => scene.focusedZoneId)

// Update listener position periodically
setInterval(() => {
  soundManager.updateListener(camera.position.x, camera.position.z, camera.rotation.y)
}, 100)
```

## Keyboard Shortcuts

| Key | Context | Action |
|-----|---------|--------|
| `Tab` | Anywhere | Switch focus between Workshop and Activity Feed |
| `Esc` | Anywhere | Switch focus between Workshop and Activity Feed |
| `1-6` | Not in input | Switch to session 1-6 (shown in UI) |
| `Q-Y` | Not in input | Switch to session 7-12 (extended) |
| `A-H` | Not in input | Switch to session 13-18 (extended) |
| `Z-N` | Not in input | Switch to session 19-24 (extended) |
| `Alt+key` | Anywhere | Switch to session (works in inputs) |
| `0` or `` ` `` | Not in input | All sessions / overview |
| `Alt+0` or `Alt+`` | Anywhere | All sessions / overview |
| `Alt+N` | Anywhere | Open new session modal |
| `Alt+A` | Anywhere | Go to next session needing attention |
| `Alt+Space` | Anywhere | Expand most recent "show more" in feed |
| `Alt+R` | Anywhere | Toggle voice recording |
| `F` | Not in input | Toggle follow-active mode |
| `P` | Not in input | Toggle station panels (tool history) |
| `Alt+D` | Anywhere | Toggle dev panel |
| `D` | Not in input | Toggle draw mode |
| `Ctrl+C` | Not in input | Context-aware: copy if text selected, interrupt working session otherwise |

**Draw Mode Keys (when active):**
| Key | Action |
|-----|--------|
| `1-6` | Select color |
| `0` | Select eraser |
| `Q/E` | Decrease/increase brush size |
| `R` | Toggle 3D stacking |
| `X` | Clear all painted hexes |
| `D` or `Esc` | Exit draw mode |

Extended keybinds (QWERTY, ASDFGH, ZXCVBN) work but are not displayed in the UI.

## Build & CLI

### npm Scripts

| Script | Description |
|--------|-------------|
| `npm run dev` | Start dev server (Vite + tsx watch) |
| `npm run build` | Build both server and client |
| `npm run build:server` | Compile server TypeScript to `dist/server/` |
| `npm run build:client` | Build frontend to `dist/` |
| `npm run server` | Run server with tsx (dev) |

### CLI Commands

```bash
vibecraft                 # Start server (uses compiled JS)
vibecraft setup           # Install hook to ~/.vibecraft/hooks/, configure all 8 hooks
vibecraft --port 4000     # Custom port
vibecraft --hook-path     # Print path to source hook script (in package)
vibecraft --help          # Show help
vibecraft --version       # Show version
```

### Publishing to npm

The `prepublishOnly` script automatically runs `build:server` before publish:

```bash
npm login
npm publish
```

Users can then run:
```bash
npx vibecraft setup
npx vibecraft
```

### TypeScript Compilation

- **Frontend**: Compiled by Vite, output to `dist/`
- **Server**: Compiled by `tsc -p tsconfig.server.json`, output to `dist/server/`
- **CLI**: Checks for compiled JS first, falls back to tsx for development

## Future Work

- [x] **Subagent visualization**: Mini-Claudes spawn at portal for Task tools
- [x] **Floating context labels**: Show file paths above stations
- [x] **Response capture**: Show Claude's responses in activity feed
- [x] **Sound effects**: Synthesized audio via Tone.js (see Sound System section)
- [ ] **Session replay**: Replay events from events.jsonl
- [ ] **File system map**: 3D visualization of touched files
- [ ] **VR support**: WebXR for immersive view

---
> Source: [Nearcyan/vibecraft](https://github.com/Nearcyan/vibecraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
