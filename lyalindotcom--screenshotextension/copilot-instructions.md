## screenshotextension

> You have access to screenshot tools that can capture the user's macOS screen.

# Screenshot Anything - Extension Capabilities

You have access to screenshot tools that can capture the user's macOS screen.

## Available Tools

### `take_screenshot`
Captures the user's screen (full screen or a locked window), resizes it for efficiency, and returns the image for analysis.

- Call `take_screenshot({})` to capture the primary display
- Use `display_id` parameter (1, 2, etc.) for other monitors
- **Locked window:** Captures immediately, no countdown
- **Full screen:** Shows a 3-second countdown dialog so the user can prepare

### `view_screenshot_history`
Retrieves previously captured screenshots from the current session.

- `{ operation: 'list' }` - See available files
- `{ operation: 'view', offset: 0 }` - View the most recent screenshot
- `{ operation: 'view', offset: 1 }` - View the previous screenshot
- `{ operation: 'view', filename: '...' }` - View a specific file

### `list_windows`
Lists all currently open and visible application windows with their IDs, app names, and titles.

### `focus_window`
Locks the screenshot capture to a specific window using fuzzy search.

**Important:** Always call `list_windows` first to see available windows. When choosing a query:

- **Use the App name for accuracy** - The query matches against both app name AND window title
- If multiple windows have similar titles, use the **App name** to disambiguate
- Example: If you see `Ghostty: "Gemini - Project"` and `Google Chrome: "Gemini CLI"`, use `{ query: "Chrome" }` or `{ query: "Google Chrome" }` to target Chrome specifically

Examples:
- `{ query: "Chrome" }` - Locks to Google Chrome (matches app name)
- `{ query: "VS Code" }` - Locks to VS Code
- `{ query: "Safari geminicli" }` - Locks to Safari window with "geminicli" in title

Once locked, all `take_screenshot` calls will only capture that window.

### `clear_lock`
Removes any active window lock. After this, `take_screenshot` reverts to full screen capture.

## When to Use These Tools

1. **Visual Debugging:** When the user asks you to "look at" an error, console output, or UI glitch
2. **UI Analysis:** When the user wants feedback on a design or layout
3. **Context Gathering:** When the answer is visible on their screen (e.g., "what version is shown?")

## Privacy Guidelines

- Only take screenshots when explicitly requested or clearly implied by the user's intent
- Do not upload images to external third-party services
- Images are saved locally in a session folder for the user's reference

---
> Source: [LyalinDotCom/ScreenshotExtension](https://github.com/LyalinDotCom/ScreenshotExtension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
