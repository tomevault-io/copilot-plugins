## oculos

> You have access to **OculOS**, a local REST API that lets you read and control any desktop application through its UI Automation tree.

# OculOS — AI Agent Instructions

You have access to **OculOS**, a local REST API that lets you read and control any desktop application through its UI Automation tree.

**Base URL:** `http://127.0.0.1:7878`

---

## Core Workflow

```
1. list_windows          → find the target application (get pid / hwnd)
2. find_elements / tree  → locate the elements you need (get oculos_id)
3. interact              → act on those elements
```

> **Important:** `oculos_id` values are session-scoped. They're valid after
> the tree is built but may change if you call tree/find again. Always fetch
> the element immediately before interacting with it.

---

## Discovery

### List all windows
```
GET /windows
```
Returns: `[{ pid, hwnd, title, exe_name, rect, visible }]`

### Search for elements (preferred — fast)
```
GET /windows/{pid}/find?q=Submit&type=Button&interactive=true
GET /hwnd/{hwnd}/find?q=Search&interactive=true
```
Parameters:
- `q` — case-insensitive substring match on label or automation_id
- `type` — element type filter (Button, Edit, CheckBox, ListItem, ComboBox…)
- `interactive=true` — return only elements that have at least one action

### Full UI tree (for exploration)
```
GET /windows/{pid}/tree
GET /hwnd/{hwnd}/tree
```
Use the HWND variant when a process has multiple windows (e.g. Teams).

---

## Interactions

All interaction endpoints accept a JSON body where noted.

| Endpoint | Body | When to use |
|----------|------|-------------|
| `POST /interact/{id}/click` | — | Buttons, links, menu items |
| `POST /interact/{id}/set-text` | `{"text":"…"}` | Input fields (replaces all text) |
| `POST /interact/{id}/send-keys` | `{"keys":"…"}` | Keyboard simulation (see below) |
| `POST /interact/{id}/focus` | — | Move keyboard focus |
| `POST /interact/{id}/toggle` | — | CheckBox, ToggleButton |
| `POST /interact/{id}/expand` | — | ComboBox, TreeItem, MenuItem |
| `POST /interact/{id}/collapse` | — | ComboBox, TreeItem, MenuItem |
| `POST /interact/{id}/select` | — | ListItem, RadioButton, TabItem |
| `POST /interact/{id}/set-range` | `{"value":75}` | Slider, Spinner |
| `POST /interact/{id}/scroll` | `{"direction":"down"}` | Scroll containers |
| `POST /interact/{id}/scroll-into-view` | — | Bring element into viewport |

### Window operations
```
POST /windows/{pid}/focus    — bring to foreground
POST /windows/{pid}/close    — close gracefully
```

---

## UiElement fields

Every element returned by the API has:

| Field | Description |
|-------|-------------|
| `oculos_id` | **Use this in all /interact calls** |
| `type` | Element type: Button, Edit, Text, CheckBox, ComboBox… |
| `label` | Accessible name (what a screen reader would announce) |
| `value` | Current text content (for Edit, ComboBox, etc.) |
| `enabled` | `false` = grayed out, skip it |
| `focused` | Is this the currently focused element? |
| `actions` | **List of valid actions for this element** — only call what's listed here |
| `toggle_state` | `"On"` / `"Off"` / `"Indeterminate"` (for CheckBox) |
| `is_selected` | `true`/`false` (for ListItem, RadioButton, TabItem) |
| `expand_state` | `"Collapsed"` / `"Expanded"` / `"LeafNode"` |
| `range` | `{ value, minimum, maximum, step }` (for Slider, Spinner) |
| `automation_id` | Stable developer-assigned ID (good for search queries) |
| `help_text` | Tooltip text |
| `rect` | `{ x, y, width, height }` in screen coordinates |
| `children` | Child elements (nested) |

**Always check `actions` before interacting.** If `actions` is empty, the element is read-only.

---

## Special keys for send-keys

Use curly braces for special keys. Combine freely with regular text.

```
{ENTER}   {TAB}     {ESC}     {SPACE}   {DELETE}  {BACKSPACE}
{UP}      {DOWN}    {LEFT}    {RIGHT}   {HOME}    {END}
{PGUP}    {PGDN}    {F1}–{F12}

{CTRL+A}  {CTRL+C}  {CTRL+V}  {CTRL+X}  {CTRL+Z}  {CTRL+Y}
{ALT+F4}  {WIN+D}
```

Examples:
```
"hello world{ENTER}"
"{CTRL+A}replacement text{ENTER}"
"{CTRL+C}"
```

---

## Common patterns

### Click a button
```bash
# 1. Find the button
GET /windows/{pid}/find?q=Submit&type=Button

# 2. Click it
POST /interact/{oculos_id}/click
```

### Type into a text field
```bash
# 1. Find the input
GET /windows/{pid}/find?q=Search&type=Edit

# 2. Set text (fast, atomic)
POST /interact/{oculos_id}/set-text
{"text": "my search query"}

# 3. Submit
POST /interact/{oculos_id}/send-keys
{"keys": "{ENTER}"}
```

### Select from a dropdown
```bash
# 1. Find and expand the combo
GET /windows/{pid}/find?q=Language&type=ComboBox
POST /interact/{oculos_id}/expand

# 2. Find the option
GET /windows/{pid}/find?q=English&type=ListItem

# 3. Select it
POST /interact/{oculos_id}/select
```

### Control a multi-window app (e.g. Teams)
```bash
# 1. List windows — note all entries with the same PID but different hwnd
GET /windows

# 2. Inspect each window separately
GET /hwnd/{hwnd1}/tree
GET /hwnd/{hwnd2}/tree

# 3. Interact as normal using oculos_id from the relevant window
```

### Check/uncheck a checkbox
```bash
GET /windows/{pid}/find?q=Remember me&type=CheckBox
# Check toggle_state in response — "Off" means currently unchecked
POST /interact/{oculos_id}/toggle
```

---

## Response format

All endpoints return:
```json
{
  "success": true,
  "data": <result>,
  "error": null
}
```

On error:
```json
{
  "success": false,
  "data": null,
  "error": "Element not found in registry"
}
```

---

## App compatibility

| App type | Coverage | Notes |
|----------|----------|-------|
| Win32 native | Excellent | Full tree, all interactions |
| WPF / .NET | Excellent | Full tree, all interactions |
| Electron (Chrome, Teams, VS Code, Slack) | Good | Shallow tree but key elements exposed |
| Qt | Good | Full tree |
| UWP / WinUI (Settings, Store) | Poor | Sandboxed — limited UIA access |

---
> Source: [huseyinstif/oculos](https://github.com/huseyinstif/oculos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
