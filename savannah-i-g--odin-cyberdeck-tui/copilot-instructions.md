## odin-cyberdeck-tui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# ODIN CYBERDECK TUI

The Odin Cyberdeck TUI is a CLI based OS that leverages the Blessed Library & Python3 to create a OS-like experience. This should primarily be focused on fullscreen operations with adaptive features to scale the system when resizing windows or using smaller terminal views.

# Key Principles

- Do not use mouse, rely on keyboard input
- Do not use Emojis
- Use all available screenspace
- Applications auto-populate from the applications folder
- Each application must have a main(term) function
- resizing detection/scaling

# Blessed Library Documentation

## Core Features
- Cross-platform terminal manipulation (Windows, Mac, Linux, BSD)
- 24-bit color support with automatic fallback
- Unicode input support
- Terminal resizing detection
- Context managers for safe terminal mode changes

## Essential Terminal Object
```python
from blessed import Terminal
term = Terminal()
```

### Key Properties
- `term.width` / `term.height` - Terminal dimensions
- `term.clear` - Clear screen
- `term.home` - Move cursor to top-left
- `term.normal` - Reset all formatting

### Context Managers
- `term.fullscreen()` - Switch to alternate screen buffer
- `term.cbreak()` - Enable character-by-character input without Enter
- `term.hidden_cursor()` - Hide cursor during drawing operations
- `term.location(x, y)` - Temporarily move cursor to position

### Colors and Formatting
- Basic colors: `term.red`, `term.green`, `term.blue`, etc.
- Background colors: `term.on_red`, `term.on_green`, etc.
- Compound formatting: `term.bold_underline_green_on_yellow`
- Format strings: `term.black_on_white("text")`

### Keyboard Input
- `term.inkey(timeout=None)` - Get single keystroke
- Returns Keystroke object with:
  - `.code` - Unique keycode for special keys
  - `.name` - Human-readable name
  - `.is_sequence` - True for special keys
- Special key constants: `term.KEY_UP`, `term.KEY_DOWN`, `term.KEY_ENTER`, `term.KEY_ESCAPE`

### Screen Positioning
- `term.move(x, y)` - Move cursor to position
- `term.move_x(x)` - Move cursor horizontally
- `term.move_y(y)` - Move cursor vertically
- `term.center(text)` - Center text on current line

## Best Practices for TUI Development

1. **Always use context managers**:
   ```python
   with term.fullscreen(), term.cbreak(), term.hidden_cursor():
       # TUI code here
   ```

2. **Handle terminal resizing**:
   - Check `term.width` and `term.height` in drawing loops
   - Adapt layout dynamically

3. **Keyboard input patterns**:
   ```python
   key = term.inkey(timeout=0.1)  # Non-blocking with timeout
   if key.code == term.KEY_ESCAPE:
       break
   ```

4. **Drawing efficiently**:
   - Use `term.location(x, y)` for positioning
   - Print with `end=''` to avoid extra newlines
   - Clear screen before redrawing: `print(term.clear, end='')`

5. **Color compatibility**:
   - Blessed automatically handles color capability detection
   - Use high-level color names for better compatibility
   - Provide fallbacks for monochrome terminals

## Application Development Guidelines

### Required Application Structure
```python
def main(term):
    # Don't use context managers since parent ODIN system handles terminal state
    # Application code here
    # Must accept terminal object as parameter
    pass
```

### CRITICAL: Terminal State Management
**DO NOT use Blessed context managers in applications:**
- Never use `with term.fullscreen()`, `term.cbreak()`, or `term.hidden_cursor()`
- The parent ODIN desktop system already manages terminal state
- Using context managers in applications causes screen clearing issues and terminal history artifacts
- Applications should draw directly using `print(term.home + term.clear, end='')` for screen clearing

### Screen Clearing in Applications
```python
# CORRECT - Use this pattern for clearing screen
print(term.home + term.clear, end='')

# INCORRECT - Never use this in ODIN applications
with term.location():
    print(term.clear, end='')
```

### Common Patterns
- Event loops with `term.inkey(timeout=0.1)`
- ESC key for exit/back navigation
- Arrow keys for menu navigation
- Enter key for selection/confirmation
- Use status bars for help text and controls
- Use `print(term.home + term.clear, end='')` for reliable screen clearing

### Terminal Safety
- Let parent ODIN system handle terminal state management
- Handle terminal interrupts gracefully
- Clear screen properly before exiting: `print(term.home + term.clear, end='')`

Blessed Documentation: https://blessed.readthedocs.io/en/latest/index.html

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/savannah-i-g) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
