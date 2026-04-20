## inspekt

> Development guidelines for AI-assisted development on the Inspekt project.

# Claude Development Guide for Inspekt

Development guidelines for AI-assisted development on the Inspekt project.

---

## Running CLI Commands

**Always use `inspekt` directly, never `python -m inspekt`.**

```bash
# Correct
inspekt inspected css --help

# Wrong - don't do this
python -m inspekt inspected css --help
```

---

## Proactive Command Execution

When explaining shell commands to the user, **always offer to execute them**:

> "Would you like me to run these commands for you?"

This applies to documentation servers, file operations, build/test commands, git operations, etc.

**Exception**: Don't offer to run destructive commands without explicit user confirmation.

---

## Inspekt MCP Tools (CRITICAL)

**ALWAYS use Inspekt MCP tools for ANY browser interaction.** Never invent alternatives.

### Golden Rules

1. **NEVER use AppleScript, osascript, or Automator** for browser automation
2. **NEVER use `screencapture`, `screenshot`, or system screenshot tools** - use `mcp__inspekt__take_screenshot`
3. **NEVER use `curl` or `wget` to fetch web pages** - use Inspekt MCP tools
4. **NEVER try to control Safari, Chrome, or Firefox directly** via shell commands
5. **NEVER use WebFetch for pages that need JS rendering or authentication** - Inspekt has access to the user's actual browser session

### When to Use What

| Task | ✅ Use This | ❌ NOT This |
|------|------------|-------------|
| Screenshot | `mcp__inspekt__take_screenshot` | `screencapture`, AppleScript |
| Navigate | `mcp__inspekt__navigate_to_url` | `open -a Safari`, AppleScript |
| Click element | `mcp__inspekt__click_element` | AppleScript, osascript |
| Type text | `mcp__inspekt__type_text` | AppleScript keyboard events |
| Press keys | `mcp__inspekt__press_keys` | AppleScript, osascript |
| Get page content | `mcp__inspekt__extract_article` | curl, wget, WebFetch |
| Run accessibility audit | `mcp__inspekt__run_axe` | External tools |
| Execute JavaScript | `mcp__inspekt__execute_javascript` | Browser DevTools manually |

### Available MCP Tools (33 total)

**Navigation:**
- `navigate_to_url` - Go to a URL
- `go_back` - Browser back button
- `go_forward` - Browser forward button
- `reload_page` - Refresh the page
- `scroll_to_top`, `scroll_to_bottom` - Scroll to edges
- `page_up`, `page_down` - Scroll by viewport

**Interaction:**
- `click_element` - Click by CSS selector
- `type_text` - Type into focused element
- `paste_text` - Paste text (faster for long text)
- `press_keys` - Keyboard shortcuts (Tab, Enter, Ctrl+A, etc.)

**Inspection:**
- `get_page_info` - URL, title, viewport size
- `take_screenshot` - Capture viewport, full page, or element
- `get_selected_text` - Get user's text selection

**Extraction:**
- `extract_links` - All links on page
- `extract_outline` - Heading hierarchy
- `extract_page_info` - Meta tags, Open Graph
- `extract_article` - Main content (Readability)

**AI-Powered:**
- `summarize` - AI article summary
- `describe` - AI page description for screen readers
- `ask` - Ask questions about the page
- `do` - Natural language actions ("click the login button")
- `index_page` - Index page for AI queries
- `md_link` - Get markdown link for current page

**Accessibility:**
- `run_axe` - axe-core WCAG audit
- `check_autocomplete` - Form field autocomplete check

**Network:**
- `get_network_requests` - Performance API network data
- `get_har` - Full HAR data (requires DevTools open)

**Storage:**
- `get_cookies`, `set_cookie` - Cookie management

**Console:**
- `get_console_logs` - Browser console messages
- `clear_console_logs` - Clear console buffer

### Troubleshooting

If an MCP tool fails:
1. Check bridge connection: `mcp__inspekt__get_page_info`
2. Verify browser extension is installed and enabled
3. Make sure `inspekt start` is running (check with `inspekt status`)

**Never fall back to AppleScript or shell hacks - ask the user to check the connection instead.**

---

## Browser Debugging

Use `inspekt console` commands autonomously instead of asking users to check their browser console.

---

## Browser VM Development

For the full VM deployment workflow, invoke the `/vm` skill.

**Quick reference — what to do after file changes:**

| Changed | Command |
|---------|---------|
| Python code (`inspekt/`) | Nothing (hot-reload in dev mode) |
| `control-server.py` | `make vm-restart-control` |
| `terminal-server.py` | `make vm-restart-terminal` |
| `control-panel.html` | `make vm-restart` |
| Chrome extension | `make vm-restart-chromium` |
| `proxy-scripts/*` | `make vm-restart-proxy` |
| Dockerfile / supervisord / entrypoint | `make vm-rebuild` |

**Key ports:** Control server = **8888**, noVNC = 6080, Terminal WS = 8889, CDP = 9222, mitmproxy = 8080

- Always use `get_bridge_port()` from `inspekt/config.py`, never hardcode ports
- Dev mode auto-enables when running from source repo

See [docs/development/vm.md](docs/development/vm.md) for full documentation.

---

## Command Palette Output Modes

Commands launched from the VM command palette (Cmd+K) use one of three output modes:

| Mode | When to use | How it works |
|------|-------------|--------------|
| **toast** | Fire-and-forget (no output) | Calls API, shows brief notification |
| **panel** | Structured output to scan | Calls API, shows flyout panel |
| **terminal** | Long output, interactive, or needs flags | Opens terminal, types and runs the command |

Defaults: Navigation → `toast`, everything else → `panel`. Override in `OUTPUT_OVERRIDES` in `control-panel.html`. Commands needing user input before execution go in `PARAM_PROMPTS`.

---

## Control Panel Popout Pattern

URL bar dropdowns (page-info, plugin dropdown) share a common architecture:

- **CSS:** `.url-bar-popout` base class (glassmorphism) + specific class (`.page-info-popout`, `.plugin-dropdown`)
- **JS:** `openPopout(panelId, triggerId, closeFn)` / `dismissActivePopout()` — unified manager ensures only one popout is open, with shared outside-click + Escape handling
- **Adding a new popout:** Create the HTML panel with `class="url-bar-popout your-popout"`, then call `openPopout()` in your open function and `dismissActivePopout()` in your close function. Zero boilerplate for dismissal logic.

---

## VNC Keyboard: Shift+Tab Fix

noVNC on Mac strips the Shift modifier from Tab during VNC protocol encoding. This is a known noVNC bug that cannot be fixed via configuration, x11vnc flags, or iframe workarounds.

**How it works:**
1. noVNC consumes the Tab keydown internally (never propagates to JS)
2. But the Tab **keyup** DOES propagate to the canvas
3. We track Shift state via keydown/keyup on the RFB canvas
4. When Tab keyup arrives while Shift is held, we send `XK_ISO_Left_Tab (0xFE20)` via `rfb.sendKey()` — the standard X11 keysym for backward tab

**Key code in control-panel.html:** Search for `setupShiftTabFix`

**Final solution:** Monkey-patch noVNC's keyboard event handler to intercept Tab when Shift is tracked. Replace the bound `_eventHandlers.keydown` listener (not just the method — the actual registered event listener must be swapped).

**What DOESN'T work (and why):**
- `e.shiftKey` on the Tab event → always `false` (noVNC strips it)
- xdotool via subprocess/os.system → works from `docker exec` but fails from control server's Python process (unknown X11 threading issue)
- CDP `Input.dispatchKeyEvent` → synthetic events don't trigger browser Tab navigation
- `rfb.sendKey()` with Shift+Tab keysyms → noVNC also sends its own broken version, causing double events
- Overriding `rfb._keyboard._handleKeyDown` directly → the old bound function is still the registered listener
- Two backward tabs to undo the forward → works but causes visible flicker
- **Working: monkey-patch `_eventHandlers.keydown`** → swap the actual listener, send `rfb.sendKey(0xFE20)` (ISO_Left_Tab), block original handler

**Safe on all platforms:** The fix only activates when Shift is held during Tab keyup. On Windows where noVNC handles Shift+Tab correctly, the fix doesn't interfere.

---

## VNC Embedding: Direct RFB (No iframe)

The VNC display is embedded directly via noVNC's `RFB` class — **no iframe**. The `connectVNC()` function dynamically imports `RFB` from `/core/rfb.js` and creates an instance on `#vncContainer`.

**Why no iframe:** Browsers swallow keyboard events (including Shift+Tab) for iframe focus management. No JavaScript workaround exists. Direct RFB embedding gives full keyboard control.

**Key patterns:**
- `rfb` global variable holds the RFB instance
- `rfb.focus()` / `rfb.blur()` for keyboard focus
- `rfb.sendKey(keysym, code, down)` for sending key events
- `rfb._keyboard.ungrab()` / `.grab()` to temporarily disable noVNC's keyboard handler
- Canvas events are intercepted via capture-phase listeners in `_setupCanvasEventInterceptors()`
- Reconnection with exponential backoff on disconnect

---

## VM Data Persistence

Plugin data (`data.db`) is stored in a named Docker volume `inspekt-vm-data` mounted at `/root/.config/inspekt/`. This survives container restarts and rebuilds.

To reset all VM data: `docker volume rm inspekt-vm-data`

---

## Plugin Autorun

Autorun plugins execute via two complementary paths:

1. **Primary (bridge):** Content script sends `browser_info` on WebSocket `onopen` → bridge triggers `_run_autorun_plugins()` — works on page refresh
2. **Backup (control panel):** `updateCurrentUrl()` detects URL change → `triggerAutorunPlugins()` calls API — catches link-click navigation missed by path 1

Debounce (3s same-URL window) prevents double-execution when both paths fire.

---

## Window Message Bridge

The extension uses `window.postMessage()` to bridge MAIN world scripts to extension APIs.

See [docs/development/bridge.md](docs/development/bridge.md) for implementation details.

---

## CLI Formatting Rules

- **Icons:** Use `icons.py` helpers (`get_icon`, `get_indicator`, `get_status_icon`, `get_action_icon`). Never hardcode Unicode.
- **Messages:** Use `table.py` functions (`print_warning`, `print_hint`, `print_error`, `print_success`)
- **Tables:** Always use the `Table` class from `table.py` (see below)

---

## Tables

Always use the `Table` class from `table.py` for tabular output. Never manually construct tables with box-drawing characters.

```python
from inspekt.app.cli.table import Table

table = Table(columns=["Rule", "Description", "Count"], border_color="bright_black")
table.add_row(["rule-1", "Some description", "5"])
table.print()
```

**Why?** Manual table construction leads to color bleeding issues where ANSI colors from cell content leak into border characters. The `Table` class handles proper styling of all borders.

---

## Tips and Hints

There are two functions for showing tips to users:

### Single Inline Hint: `print_hint()`

Use `print_hint()` from `table.py` for **contextual hints during command execution**:

```python
from inspekt.app.cli.table import print_hint
print_hint("Use `--verbose` for more details")
```

This shows a blue lightbulb icon with properly wrapped text. Use for:
- Suggesting a flag when something goes wrong
- Explaining what to do next
- Providing context about a result

### Tips Section: `_print_tips_section()`

Use `_print_tips_section()` from `inspection.py` for **multiple tips at the end of a command**:

```python
from inspekt.app.cli.inspection import _print_tips_section

tips = [
    ("--verbose", "Show detailed breakdown", "`inspekt a11y --verbose`"),
    ("--show-badges", "Visualize issues on page", None),
]
_print_tips_section(tips)
```

This shows a "TIPS" header with bullet points. Each tip is formatted as:
"Use `flag` to description (e.g. example)"

Tips wrap properly on narrow terminals.

### Never Do This

```python
# ❌ Don't use raw emoji or manual formatting
click.secho("  💡 ", fg="yellow", nl=False)
click.echo("Some tip")

# ❌ Don't use plain "Tip:" text
click.secho("Tip: Do something", fg="yellow")

# ✅ Always use the helper functions
print_hint("Do something")
```

---

## Adding New Action Types

Update these files:

- `inspekt/domain/recording.py` — Add to `ActionType` Literal
- `inspekt/scripts/record_events.js` — Event handler
- `inspekt/scripts/replay_step.js` — Replay handler
- `inspekt/app/cli/formatting.py` — Color mapping in `action_colors`
- `inspekt/app/cli/icons.py` — Icon mapping in `ACTION_ICONS`
- `inspekt/scripts/replay_visual.js` — Sound in `playForAction()`
- `inspekt/app/cli/record.py` — Tutorial `sample_steps` and `action_descriptions` dicts

---

## Adding Pre-Recording Hints/Warnings

Update these files:

- `inspekt/scripts/record_events.js` — Detection function + include in start response
- `inspekt/app/cli/record.py` — Formatter function + add to `display_pre_recording_hints()`

Use blue lightbulb for hints, yellow `⚠ Warning:` for potential issues.

---

## Popover CSS Build Process

The accessibility popover CSS is split into modular files for development but minified for production.

### CSS Source Files

Located in `inspekt/scripts/axe-popover/`:

| File | Purpose |
|------|---------|
| `tokens.css` | Design tokens (colors, spacing, fonts) |
| `base.css` | Popover base structure and positioning |
| `nav.css` | Navigation bar styles |
| `content.css` | Content sections, details, checks |
| `badges.css` | Badge base styles (interactive states only) |
| `animations.css` | Entry/exit animations |
| `themes.css` | Dark mode support |

### Building CSS

After modifying any CSS file, rebuild and update the JS files:

```bash
make build-css
# or
python scripts/build_popover_css.py --apply
```

This minifies the CSS and updates:
- `inspekt/scripts/run_axe.js` → `getPopoverCSS()`
- `inspekt/scripts/run_ibm.js` → `getIbmPopoverCSS()`

### Badge Styles

Badge styles (`.inspekt-badge` with impact colors) are **not** in the modular CSS files. They are defined inline in both `run_axe.js` and `run_ibm.js` because they require extensive `!important` declarations to override page styles.

If you need to update badge colors or sizing, modify:
- `run_axe.js` → inline styles around line 800
- `run_ibm.js` → `injectBadgeStyles()` function

### Dev Mode CSS

For faster iteration, use the `--dev-css` flag:

```bash
inspekt axe --show-badges --interactive --dev-css
```

This loads CSS from `http://localhost:3456/` instead of the inline minified version.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/roelvangils) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
