## brain-edit-slides

> Add a Canva-style visual Edit Mode to any HTML presentation/slide deck. Use this skill EVERY TIME you create, generate, or finish building an HTML presentation, slide deck, or מצגת for the user - inject the editor automatically as the final step. Also use when the user asks to "make my deck editable", "add edit mode", "הוסף מצב עריכה", "תעשה שאוכל לערוך את המצגת", or wants to move/edit/add elements in an existing HTML deck.


# Deck Edit Mode

Adds an in-browser visual editor to any self-contained HTML slide deck:
an "✏️ עריכה" button that lets the user drag elements, edit text inline,
paste screenshots (Cmd+V), add images/blur/shapes/textboxes, recolor text,
nudge with arrow keys, undo/redo, autosave - and save the edited file.
No server, no dependencies. Everything stays inside the single HTML file.

## When to run

1. **Automatically** - after you create or significantly rebuild any HTML
   presentation for the user, inject the editor as the last build step.
2. **On request** - when the user asks to make an existing deck editable.

## How to inject

```bash
python3 <skill-dir>/scripts/inject_editor.py path/to/deck.html
```

The script is idempotent: running it again upgrades the editor in place
(it replaces the previous editor block, never duplicates). It appends the
editor CSS before the last `</style>` and the editor JS before `</body>`.
Always keep a backup copy of the deck before the first injection.

## Deck compatibility - IMPORTANT, adapt when needed

The editor expects the common Claude-generated deck shape:

- A fixed-resolution stage container with `id="deck"` (default 1920x1080),
  scaled to fit the window with `transform: scale(...)`.
- Slides as `.slide` elements where the visible one has class `.active`.
- Optionally a global `show(n)` function for navigation and entry
  animations on `.anim` elements.

Decks are built by many different skills, so BEFORE injecting, read the
target HTML and check the structure. If it differs, adapt - the constants
live at the top of `scripts/editor.js`:

| Deck difference | What to adapt |
|---|---|
| Stage container has another id/class | In `editor.js`, change `getElementById('deck')` to match, and the two `1920` / `1080` design-resolution constants (`scale()`, guides CSS in `editor.css`) to the deck's design size |
| Slides aren't `.slide` / `.active` | Update the `.slide` / `.active` selectors in `editor.js` (`activeSlide`, `eligible`, export) |
| Full-bleed backgrounds / wrappers get selected by mistake | Add their selectors to the `DENY` list |
| A text-like element refuses double-click editing | Add its selector to the `TEXTY` list |
| Animated number counters (`[data-count]`-style) | The editor already syncs `data-count` on edit and bakes values on export; if the deck uses a different attribute, mirror that logic |
| Single-page (non-slide) presentation | Treat the whole page container as one "slide": give it class `slide active` or adapt `activeSlide()` |

After injecting, ALWAYS verify: open the file (or screenshot it headless),
confirm the deck still renders identically in present mode, the ✏️ button
appears, and edit mode opens. If anything looks broken, restore the backup.

## What the user gets (tell them after injecting)

- ✏️ עריכה button (bottom-right) toggles edit mode
- Click = select · drag = move (center snap-guides, Alt disables) ·
  double-click = edit text (Escape cancels)
- Cmd+C / Cmd+V copy-paste elements · paste a screenshot directly ·
  drag image files from the desktop · Cmd+D duplicate
- Toolbar: add image / blur / shape / textbox · ⤴ select parent group ·
  z-order · center H/V · font size · color swatches (whole element, or a
  selected word while editing)
- Arrow keys nudge 1px (Shift = 10px) · Cmd+Z undo · Cmd+Shift+Z redo
- Autosave to the browser; a restore bar appears if work was lost
- 💾 שמור / Cmd+S downloads `<name>-edited.html` - the user replaces the
  original file with it. Backgrounds, fonts and numbers are preserved
  (URLs are absolutized and counters baked at export)

## Auto-injection for every future deck

On first install, offer the user to add this line to their project's
CLAUDE.md so the behavior survives across sessions:

```
After creating any HTML presentation, inject Edit Mode:
python3 <skill-dir>/scripts/inject_editor.py <deck.html>
```

---
> Source: [Brain-ai-biz/Brain-edit-slides](https://github.com/Brain-ai-biz/Brain-edit-slides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
