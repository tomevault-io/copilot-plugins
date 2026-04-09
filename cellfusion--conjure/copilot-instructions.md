## conjure

> This file is a guide for Claude Code and AI assistants to understand and work efficiently on this project.

# CLAUDE.md

This file is a guide for Claude Code and AI assistants to understand and work efficiently on this project.

## CRITICAL: Mandatory Working Principles

**THESE RULES ARE ABSOLUTE AND MUST BE FOLLOWED WITHOUT EXCEPTION:**

### 1. NO GUESSING OR ASSUMPTIONS
- **NEVER** provide solutions based on assumptions or guesses
- **NEVER** say "this should work" or "probably" without verification
- **ALWAYS** verify facts from official documentation before suggesting solutions
- If you don't know something with certainty, explicitly state: "I don't know for certain, let me research this"

### 2. VERIFY ALL SOURCES
- **BEFORE** citing any documentation, API, or code example:
  1. Fetch the actual source document
  2. Verify the information exists and is accurate
  3. Check the version/date of the documentation matches the project dependencies
- **NEVER** cite sources you haven't actually read
- **ALWAYS** include direct links to sources in your responses

### 3. TEST BEFORE SUGGESTING
- **WHEN POSSIBLE**, test solutions before suggesting them:
  1. Build the code to verify it compiles
  2. Check for compilation errors or warnings
  3. Verify API methods exist in the actual crate documentation
- **IF TESTING IS NOT POSSIBLE**, explicitly state this limitation
- **NEVER** suggest code changes that use non-existent APIs

### 4. INCREMENTAL VERIFICATION
- Make ONE change at a time when debugging
- Verify each change works before proceeding
- Document what was tried and what failed
- Keep track of what has been eliminated as potential causes

### 5. ADMIT IGNORANCE
- If you don't understand how something works, say so
- Don't make up explanations to fill knowledge gaps
- Suggest concrete steps to learn the answer (read docs, check source code, etc.)
- It's better to say "I don't know" than to waste time with wrong solutions

### 6. CONSEQUENCES OF VIOLATION
Violating these rules wastes the user's time and erodes trust. Every guess that fails:
- Costs development time
- Increases frustration
- Delays actual progress
- Damages confidence in AI assistance

**If you cannot follow these principles, explicitly decline the task and explain why.**

## Project Overview

**Name**: conjure  
**Type**: Zellij Plugin (WebAssembly)  
**Language**: Rust  
**Purpose**: A plugin to magically send commands to panes using tab and pane names (CLI + Interactive UI)

### Why "conjure"?

**conjure** = "to summon, to make appear as if by magic"

Chosen after analyzing existing awesome-zellij plugin naming patterns:
- 🎩 **Magical experience**: Magically send commands to any pane
- ✨ **Intuitive**: "Summon panes and send commands" concept is clear
- 🎯 **Unique**: Follows pattern of short, catchy names like `ghost`, `grab`, `room`
- 📦 **Memorable**: Short, easy to pronounce and type
- 🚀 **Differentiated**: Doesn't conflict with existing plugins, evokes functionality

CLI commands are also simple:
```bash
# Send command to a pane
zellij action pipe --plugin conjure --name send_command \
  -- '{"pane_name":"terminal","text":"npm run build"}'

# List all panes (returns JSON)
zellij action pipe --plugin conjure --name list_panes

# Search panes by name/process/tab (fuzzy match, AND condition)
zellij action pipe --plugin conjure --name search_panes \
  -- '{"pane_name":"terminal"}'
zellij action pipe --plugin conjure --name search_panes \
  -- '{"process_name":"nvim","tab_name":"Editor"}'

# Without config.kdl - use full path
zellij action pipe --plugin file:~/.config/zellij/plugins/conjure.wasm --name send_command \
  -- '{"pane_name":"terminal","text":"npm run build","send_enter":true}'
```

### CLI Response Format

All CLI pipe commands return JSON responses via `cli_pipe_output`:

```json
// Success (list_panes / search_panes)
{"success":true,"data":[{"pane_id":3,"pane_title":"terminal","tab_name":"Editor","tab_position":0,"is_focused":true,"process_name":"bash"}]}

// Success (send_command)
{"success":true}

// Error
{"success":false,"error":"Pane 'xxx' not found. Available tabs: [...]"}
```

### Problem and Solution

**Problem**: 
- Sending commands to specific panes requires numeric pane_id
- pane_id only visible via `$ZELLIJ_PANE_ID` environment variable
- No interactive pane selection and command sending
- Less user-friendly compared to tmux's `send-keys -t`

**Solution**:
1. **CLI**: Specify by tab and pane names via pipe
2. **Interactive UI**: Fuzzy finder for pane search → selection → command input
3. **Presets**: Save and execute frequently-used commands
4. **History**: Manage recently used panes and commands

## Tech Stack

- **Language**: Rust
- **Target**: wasm32-wasip1
- **Dependencies**:
  - `zellij-tile` (0.43.1): Zellij plugin development SDK
  - `serde`, `serde_json`: JSON serialization
  - `chrono`: Timestamp management (for history)
  - `fuzzy-matcher`: Fuzzy search (future)

## Architecture

### Two Operation Modes

#### 1. CLI Mode (via Pipe)
```
User → zellij action pipe --plugin conjure --name <action> → Plugin
                                                               ↓
                                                    Route by pipe name
                                               ┌──────────┼──────────┐
                                          list_panes  search_panes  send_command
                                               ↓          ↓              ↓
                                          build_      fuzzy_match   write_chars_
                                          display_    + filter      to_pane_id
                                          panes()                       ↓
                                               ↓          ↓        Save to History
                                               └──────────┴──────────┘
                                                          ↓
                                                  cli_pipe_output (JSON)
```

#### 2. UI Mode (Floating Pane)
```
User → Keybinding → LaunchOrFocusPlugin → Tab UI
                                             ↓
                                    ┌────────────────┐
                                    │ [Panes] History Presets │ ← Tab bar
                                    └────────────────┘
                                            ↓
                                    Tab Content Area
                                    ├─ Panes: Select pane → Command Input
                                    ├─ History: Execute or delete
                                    └─ Presets: Execute preset
                                            ↓
                                  write_chars_to_pane_id
                                            ↓
                                      Save to History
```

### Keyboard Shortcuts (UI Mode)

**Global (All Tabs):**
- `TAB` - Switch to next tab (Panes → History → Presets → Panes)
- `Shift+TAB` - Switch to previous tab (Panes → Presets → History → Panes)
- `Esc` / `Ctrl+p x` - Close UI

**Panes Tab:**
- `↑↓` - Navigate panes
- `Enter` - Select pane and enter command
- Type characters - Search/filter panes

**History Tab:**
- `↑↓` - Navigate history entries
- `Enter` - Execute command from history
- `d` - Delete selected history entry

**Presets Tab:**
- `↑↓` - Navigate presets
- `Enter` - Execute preset command

**Command Input Mode:**
- Type - Enter command
- `Enter` - Execute command
- `Backspace` - Delete character
- `Esc` - Cancel and close UI

### Core Components

1. **State Management** (`src/state.rs`)
   - Manage state from TabUpdate/PaneUpdate events
   - Manage UI state (selected index, search query, etc.)
   - Manage history and presets
   - Helper methods: `find_pane_by_names()`, `find_pane_info()`

2. **UI Layer** (`src/ui/`)
   - `pane_selector.rs`: Pane selection screen with fuzzy search
   - `command_input.rs`: Command input screen
   - `history_list.rs`: Command history browser
   - `preset_list.rs`: Preset command browser
   - `render.rs`: Shared rendering utilities

3. **Command Execution** (`src/executor.rs`)
   - Parse JSON payload from CLI
   - Execute send_command action
   - Execute list_panes and search_panes queries
   - Error handling with detailed messages

4. **History & Presets** (`src/history.rs`, `src/presets.rs`)
   - File I/O with graceful error handling
   - Data structure management
   - History: Auto-save after each command, max 100 entries (FIFO)
   - Presets: Load from `/data/conjure_presets.json` at startup

5. **Command Types** (`src/commands.rs`)
   - JSON payload structures (`SendCommandPayload`, `SearchPanesPayload`)
   - CLI response types (`CliResponse<T>`, `PaneOutput`)
   - Serialization/deserialization

## Important APIs

### Zellij Plugin API

```rust
// Subscribe to events (in load() method)
subscribe(&[
    EventType::TabUpdate,
    EventType::PaneUpdate,
    EventType::Key,        // For UI mode
]);

// Request permissions (in load() method)
request_permission(&[
    PermissionType::ReadApplicationState,
    PermissionType::WriteToStdin,
    PermissionType::ReadCliPipes,
]);

// Handle pipe messages (CLI commands, plugin-to-plugin, keybindings)
fn pipe(&mut self, pipe_message: PipeMessage) -> bool {
    match pipe_message.source {
        PipeSource::Cli(pipe_id) => {
            // Handle CLI pipe messages
            if let Some(payload) = pipe_message.payload {
                // Parse JSON payload and execute command
            }
        }
        PipeSource::Plugin(plugin_id) => {
            // Handle plugin-to-plugin messages
        }
        PipeSource::Keybind => {
            // Handle keybinding messages
        }
    }
    false // Return true if should render
}

// Send text to pane
write_chars_to_pane_id(&text, pane_id);

// Send Enter key (CR = 0x0D = 13)
write_to_pane_id(vec![0x0D], pane_id);

// Send response back to CLI caller (requires ReadCliPipes permission)
cli_pipe_output(&pipe_id, &json_string);
```

## Development Guidelines

### Build and Test

```bash
# Add wasm target (first time only)
rustup target add wasm32-wasip1

# Build
cargo build --target wasm32-wasip1 --release

# Install
cp target/wasm32-wasip1/release/conjure.wasm ~/.config/zellij/plugins/

# Grant permissions (first time)
zellij plugin -- file:$HOME/.config/zellij/plugins/conjure.wasm

# Test UI mode (after keybinding setup)
# Ctrl + p, c

# Test CLI mode (after adding plugin to config.kdl)
zellij action pipe --plugin conjure --name send_command \
  -- '{"action":"send_command","tab_name":"Editor","pane_name":"terminal","text":"echo hello","send_enter":true}'

# Test CLI mode (without config.kdl - using full path)
zellij action pipe --plugin file:$HOME/.config/zellij/plugins/conjure.wasm --name send_command \
  -- '{"action":"send_command","tab_name":"Editor","pane_name":"terminal","text":"echo hello","send_enter":true}'
```

### Common Implementation Patterns

#### Fuzzy Matching

```rust
fn fuzzy_match(text: &str, pattern: &str) -> Option<i32> {
    if pattern.is_empty() {
        return Some(0);
    }
    
    let text = text.to_lowercase();
    let pattern = pattern.to_lowercase();
    
    let mut score = 0;
    let mut text_idx = 0;
    
    for pattern_char in pattern.chars() {
        loop {
            if text_idx >= text.len() {
                return None;
            }
            if text.chars().nth(text_idx) == Some(pattern_char) {
                score += 1;
                text_idx += 1;
                break;
            }
            text_idx += 1;
        }
    }
    
    Some(score)
}
```

## Troubleshooting

### Issue: UI not displaying

**Causes**:
- `render()` not being called
- `update()` returning `false`
- Plugin not launched in floating mode

**Solution**:
```rust
fn update(&mut self, event: Event) -> bool {
    match event {
        Event::Key(_) if self.ui_mode != UiMode::Hidden => {
            // Always return true when handling key events
            true
        }
        _ => false
    }
}
```

### Issue: Pane not found

**Debug method**:
```rust
eprintln!("Searching for tab='{}' pane='{}'", tab_name, pane_name);
eprintln!("Available tabs: {:?}", self.tabs.keys());
eprintln!("Available panes: {:?}", self.pane_lookup.values());
```

## Reference Resources

### Essential Documentation

1. **Zellij Plugin Tutorial**
   https://zellij.dev/tutorials/developing-a-rust-plugin/

2. **zellij-tile API Reference**
   https://docs.rs/zellij-tile/latest/zellij_tile/

3. **Similar Plugins**
   - [zellij-send-keys](https://github.com/atani/zellij-send-keys): Send via pane_id
   - [zellij-pane-picker](https://github.com/shihanng/zellij-pane-picker): Pane selection UI
   - [harpoon](https://github.com/Nacho114/harpoon): Quick pane navigation
   - [zellij-playbooks](https://github.com/yaroslavborbat/zellij-playbooks): Command books

## Development Checklist

### Phase 1: Basic CLI ✅ COMPLETED
- [x] Project setup (zellij-tile 0.43.1, dependencies)
- [x] Basic plugin structure with pipe() method
- [x] TabUpdate/PaneUpdate state management
- [x] Name-based pane search
- [x] JSON payload parsing in pipe() method
- [x] CLI command sending via write_chars_to_pane_id

### Phase 2: Basic UI ✅ COMPLETED
- [x] Pane selector UI rendering
- [x] Keyboard navigation
- [x] Fuzzy search
- [x] Command input UI
- [x] Enter key handling

### Phase 3: History & Presets ✅ COMPLETED
- [x] Save/load history (auto-save after each command)
- [x] Preset configuration file (`/data/conjure_presets.json`)
- [x] Tab-based UI with TAB/Shift+TAB navigation (like session-manager)
- [x] History tab: reverse chronological display, execute, delete support
- [x] Presets tab: multiline display with targets, execute support
- [x] Unified tab view rendering with active tab highlighting
- [x] History persistence (max 100 entries, FIFO)

### Phase 4: Integration & Testing
- [ ] Keybinding configuration
- [ ] Error handling
- [ ] Documentation
- [ ] Demo GIFs

## For AI Assistants

When generating code:

1. **Use English for code and commits**
   - All source code comments must be in English
   - All commit messages must be in English
   - Documentation can be in English or Japanese as appropriate

2. **Reference specification (docs/SPECIFICATION.md)**
   - Check feature requirements
   - Follow data structures
   - Adhere to UI design

3. **Implement incrementally**
   - Follow phases in order
   - Verify before proceeding
   - Don't create everything at once

4. **Reference existing plugin patterns**
   - zellij-send-keys: Pipe handling
   - zellij-pane-picker: UI implementation
   - default-plugins: Event handling

5. **Thorough error handling**
   - File I/O always uses `Result`
   - JSON parsing with error handling
   - Clear messages to users

6. **Add debug information**
   - `eprintln!` for important operations
   - Log state transitions
   - Detailed error information

7. **Documentation file organization**
   - Save documentation files under `docs/` directory
   - `CLAUDE.md` and `README.md` should be in the root directory
   - Keep project documentation organized by topic

For questions or suggestions:
- Explain why you chose that approach
- Present alternatives if available
- Make tradeoffs clear

## File Locations and Formats

All data files are stored in the Zellij data directory (usually `~/.local/share/zellij/` or as specified by `$ZELLIJ_DATA_DIR`).

### History File: `/data/history.json`

**Auto-generated** - No manual editing needed.

Example:
```json
[
  {
    "timestamp": "2026-02-11T10:30:45Z",
    "tab_name": "Editor",
    "pane_name": "terminal",
    "pane_id": 3,
    "command": "cargo test"
  }
]
```

**Features:**
- Automatically saved after each command execution
- Maximum 100 entries (oldest removed when limit exceeded)
- Stored in reverse chronological order in UI
- Can be deleted via UI (press `d` in History List)

### Preset File: `/data/conjure_presets.json`

**User-created** - Create this file manually to define command shortcuts.

Example:
```json
[
  {
    "name": "Run Tests",
    "description": "Execute all cargo tests",
    "tab_name": "Editor",
    "pane_name": "terminal",
    "command": "cargo test --all"
  },
  {
    "name": "Git Status",
    "description": "Check repository status",
    "tab_name": null,
    "pane_name": "terminal",
    "command": "git status"
  }
]
```

**Field Descriptions:**
- `name` (required): Display name for the preset
- `description` (optional): Description shown in UI
- `tab_name` (optional): Target tab name (null = search all tabs)
- `pane_name` (required): Target pane title/name
- `command` (required): Command to execute

**Notes:**
- If `tab_name` is `null`, the plugin searches all tabs for matching `pane_name`
- If target pane doesn't exist, an error message is shown
- Preset execution also saves to history

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cellfusion)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cellfusion)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
