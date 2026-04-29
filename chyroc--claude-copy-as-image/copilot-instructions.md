## claude-copy-as-image

> Instructions for AI agents working on this codebase.

# AGENTS.md

Instructions for AI agents working on this codebase.

## Project Overview

This is a **Claude Code skill** that converts Claude's markdown responses into
beautiful Carbon/Ray.so-style PNG screenshots and copies them to the clipboard.

Users install via `npx skills add chyroc/claude-copy-as-image -y -g` and invoke
with `/copy-as-image` inside Claude Code.

## Repository Structure

```
├── skills/copy-as-image/
│   ├── SKILL.md                # Slash command definition (what Claude executes)
│   ├── scripts/
│   │   └── copy-as-image.mjs   # Core script: markdown → SVG → PNG → clipboard
│   └── package.json            # Runtime dep: @resvg/resvg-js
├── screenshot.png              # Example output for README
├── AGENTS.md                   # This file
├── README.md
└── LICENSE
```

## Architecture

### Pipeline

```
User types /copy-as-image
  → Claude recalls last substantial response
  → Writes markdown to temp file
  → Runs: node copy-as-image.mjs <tempfile>
      → parseMarkdown()    : markdown text → styled spans (color, bold)
      → wrapSpans()        : auto-wrap long lines at 100 display-columns
      → textToSvg()        : spans → SVG with gradient bg, shadow, window chrome
      → svgToPng()         : SVG → PNG via @resvg/resvg-js
      → copyToClipboard()  : PNG → system clipboard (osascript / xclip)
  → PNG also saved to /tmp/claude-screenshot-XXXX.png
```

### Key Files

**`skills/copy-as-image/SKILL.md`** — The prompt Claude follows when `/copy-as-image`
is invoked. It tells Claude to:
1. Find the last substantial response (skip one-liners, slash outputs, confirmations)
2. Write it to a temp file with a timestamp-based name
3. Run the script in a single Bash call
4. Report the result and PNG path to the user

**`skills/copy-as-image/scripts/copy-as-image.mjs`** — Pure ESM Node.js script (~360 lines).
No TypeScript, no build step. Key sections:

| Section | What it does |
|---------|-------------|
| `THEME` | One Dark color palette — card bg, text colors, traffic light colors |
| `GRADIENTS` | 7 background gradient presets (midnight, breeze, candy, sunset, meadow, ocean, aurora) |
| `isFullWidth()` / `stringDisplayWidth()` | CJK-aware character width calculation |
| `wrapSpans()` | Word wrap at configurable max columns |
| `parseMarkdown()` / `parseInline()` | Markdown → colored spans (headings, bold, code, lists, links) |
| `textToSvg()` | SVG generation with gradient, shadow filter, window chrome, text rendering |
| `findSystemFonts()` | Locate monospace + CJK font files on macOS/Linux |
| `svgToPng()` | @resvg/resvg-js render with explicit font binding |
| `copyToClipboard()` | macOS `osascript` / Linux `xclip` / `xsel` |

## Development Guide

### Running locally

```bash
cd skills/copy-as-image
npm install
echo "# Test\n\nHello **World**" | node scripts/copy-as-image.mjs
```

### Testing with Claude Code

```bash
# Install as a local skill
npx skills add ./  -y -g

# Or point Claude Code directly
claude --plugin-dir .
```

### Code Style

- Pure ESM (`.mjs`), no transpilation needed
- No TypeScript — keeps it runnable with just `node`, zero build step
- Only one runtime dependency: `@resvg/resvg-js` (WASM, no native compilation)
- All other logic is hand-written: markdown parser, SVG generator, clipboard integration

### When modifying the SVG renderer

The SVG is built as a string in `textToSvg()`. The visual layers from back to front:

1. **Background** — full-size rect with gradient fill (`url(#bg-grad)`)
2. **Card** — rounded rect with `feDropShadow` filter (dx=0, dy=16, blur=28)
3. **Title bar** — darker rect at card top with rounded top corners
4. **Traffic lights** — three circles (red #ff5f57, yellow #febc2e, green #28c840)
5. **Text content** — `<text>` elements with `<tspan>` per styled segment

To change the theme: modify `THEME` constants at the top.
To change the background: modify `ACTIVE_BG` (points to one of `GRADIENTS`).

### Font handling

Fonts are critical. The script uses `@resvg/resvg-js` with:
- `loadSystemFonts: false` — don't let resvg guess (it may pick proportional fonts)
- `fontFiles: [...]` — explicit paths to Menlo (monospace) + STHeiti/Hiragino (CJK)
- `defaultFontFamily: "Menlo"` — force monospace for all text

If text renders as question marks/tofu: the font file is missing. Check `findSystemFonts()`.
If text is not monospaced: resvg is falling back to a proportional font. Ensure `loadSystemFonts: false`.

### Clipboard integration

| Platform | Method | Notes |
|----------|--------|-------|
| macOS | `osascript -e 'set the clipboard to (read (POSIX file "...") as «class PNGf»)'` | The `«class PNGf»` AppleScript type is the only reliable CLI method for PNG clipboard |
| Linux | `xclip -selection clipboard -t image/png -i "..."` | Falls back to `xsel`. User must install one of them |

### Common issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Question marks / tofu in image | Font files not found | Check `findSystemFonts()` paths exist on the target OS |
| Text not monospaced | resvg falling back to proportional font | Ensure `loadSystemFonts: false` + `fontFiles` has Menlo/DejaVu |
| Text overflows card | Line too long, or CJK width miscounted | Check `wrapSpans()` and `isFullWidth()` |
| Chinese characters as boxes | CJK font not loaded | Add STHeiti/Hiragino/Noto Sans CJK to `findSystemFonts()` |
| Clipboard copy fails | Missing osascript (macOS) or xclip (Linux) | Platform-specific, check `copyToClipboard()` |
| `mktemp` template not expanding | SKILL.md used `mktemp XXXXXX` inside heredoc | Use `date +%s%N` for temp filenames instead |
| npm install warns `--production` | Deprecated flag | Use `--omit=dev` instead |

---
> Source: [chyroc/claude-copy-as-image](https://github.com/chyroc/claude-copy-as-image) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
